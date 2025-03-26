# Interceptor 2

2025.03.26 https://www.vulnvm.com/interceptor-2

[video]()

## Ip

192.168.5.40

## Scan

```
PORT    STATE SERVICE
21/tcp  open  ftp
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

smb 上不允许匿名登陆。

发现 wordpress http://192.168.5.40/wordpress/
