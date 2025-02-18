# HACKNOS: OS-HACKNOS-3

2025.02.x https://www.vulnhub.com/entry/hacknos-os-hacknos-3,410/

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

扫描发现http://192.168.5.40/upload.php ffuf 探测一下参数，没找到。看到 80 首页底部提示：find the Bug You need extra WebSec

在访问 http://192.168.5.40/websec/ 目录得到新的页面，gobuster 再次扫描，发现了一个文件包含的 url http://192.168.5.40/websec/log/?url=about
