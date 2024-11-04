# Convert

https://hackmyvm.eu/machines/machine.php?vm=Convert

## Ip

192.168.5.21

## Scan

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

重点来看这个 html to pdf 的功能，把自己转换出来的 pdf 保存在 kali 上，然后查看 pdf 详细信息：

```
pdfinfo 1111.pdf

Producer:        dompdf 1.2.0 + CPDF
CreationDate:    Sun Apr 28 11:03:52 2024 CST
ModDate:         Sun Apr 28 11:03:52 2024 CST
```

看到是 dompdf 1.2.0 转换生成的，经过搜索，这个 pdf 转换库，存在 RCE 漏洞 51270.py
