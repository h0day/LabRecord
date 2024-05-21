# So Simple: 1

2024-5-20 https://vulnyx.com/

difficulty: easy

## IP

192.168.5.30

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey:
|   256 e1858b7b6da26b1aed188e08a090872a (ECDSA)
|_  256 adfe7778a05770cc3368b58426a3b363 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Essex
```

直接看 80 服务，主页没看到什么信息，使用 gobuster 进行扫描：

```
http://192.168.5.30/index.html           (Status: 200) [Size: 2447]
http://192.168.5.30/index.php            (Status: 200) [Size: 72605]
http://192.168.5.30/backup               (Status: 301) [Size: 313] [--> http://192.168.5.30/backup/]
http://192.168.5.30/imagenes             (Status: 301) [Size: 315] [--> http://192.168.5.30/imagenes/]
```

http://192.168.5.30/index.php 是一个 phpinfo 页面，网站的根目录在 /var/www/html 中。

http://192.168.5.30/backup/upload.php 是一个文件上传页面，上传 php 有拦截，上传其他文件可以通过。
