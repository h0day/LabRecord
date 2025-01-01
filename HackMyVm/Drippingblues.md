# Drippingblues

2025.01.01 https://hackmyvm.eu/machines/machine.php?vm=Drippingblues

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 可以匿名登陆，其中有一个 zip 文件包 respectmydrip.zip，将其下载，解压需要密码，使用 john 进行破解，其解压密码为 072528035，解压后，其中还有一个 secret.zip 文件，但是还有密码，rockyou 也破解不出密码。

respectmydrip.txt 中的提示内容为: just focus on "drip" ，drip 好像是个用户名，目前其他的不清楚。

在看看 80 web 服务吧，扫描目录发现 http://192.168.5.39/robots.txt :

```
User-agent: *
Disallow: /dripisreal.txt
Disallow: /etc/dripispowerful.html
```

http://192.168.5.39/dripisreal.txt 中提示了 ssh 密码的获得方式：

```
hello dear hacker wannabe,

go for this lyrics:

https://www.azlyrics.com/lyrics/youngthug/constantlyhating.html

count the n words and put them side by side then md5sum it

ie, hellohellohellohello >> md5sum hellohellohellohello

it's the password of ssh
```

但是不知道 n 是多少，搞不出来。

在看看 /etc/dripispowerful.html 提示 404。经过查询，发现需要这样构造 http://192.168.5.39/index.php?drip=/etc/dripispowerful.html 从 html 源码中找到了密码：

```
password is:
imdrippinbiatch
```

index.php 中底部提示了 2 个用户 travisscott & thugger 尝试用上面的密码登陆，发现 thugger:imdrippinbiatch 能够登陆。

先获得了 user flag:

```
thugger@drippingblues:~$ cat user.txt
5C50FC503A2ABE93B4C5EE3425496521
```

进行系统枚举，没发现其他提权路径，最后选择了 pwnkit 提权：https://github.com/ly4k/PwnKit

将 PwnKit 下载到目标机器上执行后，得到了 root 权限。

```
root@drippingblues:~# cat root.txt
78CE377EF7F10FF0EDCA63DD60EE63B8
```
