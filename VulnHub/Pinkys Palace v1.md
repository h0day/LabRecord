# Pinky's Palace: v1

2024-4-29 https://www.vulnhub.com/entry/pinkys-palace-v1,225/

difficulty: Easy/Intermediate

## IP

192.168.5.22

## Scan

Open Port -> 8080,31337,64666

```
PORT      STATE SERVICE    VERSION
8080/tcp  open  http       nginx 1.10.3
|_http-title: 403 Forbidden
|_http-server-header: nginx/1.10.3
31337/tcp open  http-proxy Squid http proxy 3.5.23
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/3.5.23
64666/tcp open  ssh        OpenSSH 7.4p1 Debian 10+deb9u2 (protocol 2.0)
| ssh-hostkey:
|   2048 df02124f4c6d50276a84e90e5b65bfa0 (RSA)
|   256 0aadaac716f71507f0a8502317f31c2e (ECDSA)
|_  256 4a2de5d8ee696155bbdbaf294e54522f (ED25519)
```

只开了 3 个端口，其中有一个 squid 代理。

8080 访问后，返回 403，看样子需要我们使用 squid 代理去访问。
