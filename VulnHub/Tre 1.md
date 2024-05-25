# Tre: 1

2024-5-25 https://www.vulnhub.com/entry/tre-1,483/

difficulty: Intermediate

## IP

192.168.10.188

## Scan

Open Port -> 22,80,8082

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 991aead7d7b348809f88822a14eb5f0e (RSA)
|   256 f4f69cdbcfd4df6a910a8105defa8df8 (ECDSA)
|_  256 edb9a9d72d00f81bd399d602e5ad179f (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Tre
8082/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Tre
```

8082 对应的 nginx，gobuster 扫描不出其他目录，估计是个摆设。

gobuster 扫描 80 端口，看看有什么信息：

```
http://192.168.10.188/info.php             (Status: 200) [Size: 87812]
http://192.168.10.188/cms                  (Status: 301) [Size: 314] [--> http://192.168.10.188/cms/]
http://192.168.10.188/index.html           (Status: 200) [Size: 164]
```

/info.php 是 phpinfo 页面。

http://192.168.10.188/cms 要有 http basic 认证，尝试使用 admin:admin 认证成功，页面跳转到 http://192.168.10.188/system/login_page.php 是一个登陆页面，没有用户名和密码。

看到 CMS 的名字是 mantis
