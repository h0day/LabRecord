# Convert

https://hackmyvm.eu/machines/machine.php?vm=Convert

difficulty: Easy

## ip

192.168.5.14

## scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey:
|   256 d87a1e74a21a4074911f819b057c9af6 (ECDSA)
|_  256 289ff8ce7b5de1a7fa23c1fe00ee6324 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-title: HTML to PDF
|_http-server-header: nginx/1.22.1
```

22 端口上不允许匿名登陆，看看 80 web 上有什么信息：http://192.168.5.11/ 显示了一个网页转化为 pdf 的网站，根据经验，可能存在 SSRF 漏洞。
