# Liceo

2025.03.26 https://hackmyvm.eu/machines/machine.php?vm=Liceo

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 匿名登陆发现：

```
Hi Matias, I have left on the web the continuations of today's work,
would you mind contiuing in your turn and make sure that the web will be secure?
Above all, we dont't want intruders...
```

发现一个用户名 Matias

web 发现上传 http://192.168.5.39/upload.php 上传的文件存储在 http://192.168.5.39/uploads/ 不能上传 php 文件，但可以上传 phtml 文件，上传 webshell，拿到反弹 shell。

先拿到 user flag：

```
bash-5.1$ cat user.txt
71ab613fa286844425523780a7ebbab2
```

发现 suid 直接提权到 root：

```
bash-5.1$ /usr/bin/bash -p
bash-5.1# cd /root
bash-5.1# ls
root.txt  snap
bash-5.1# cat root.txt
BF9A57023EDD8CFAB92B8EA516676B0D
```
