# SpyderSec: Challenge

2024-5-x https://www.vulnhub.com/entry/spydersec-challenge,128/

difficulty: Intermediate

## IP

192.168.5.26

## Scan

Open Port -> 22,80

```
PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd
|_http-title: SpyderSec | Challenge
|_http-server-header: Apache
```

2 个开放端口。22 ssh banner 没什么提示信息。

80 端口是 web 服务，主页显示了找到 flag 的相关提示，源代码也没什么信息。

使用目录扫描，发现隐藏目录 http://192.168.5.26/v/
