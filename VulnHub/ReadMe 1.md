# ReadMe: 1

2025.02.22 https://www.vulnhub.com/entry/readme-1,336/

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```

web 扫描发现 http://192.168.5.40/info.php 是 phpinfo 页面，mysql 版本 mysqlnd 5.0.12-dev 。

发现 http://192.168.5.40/reminder.php 里面说密码在 txt 文件中，然后下面是一个搜索 username 的搜索框，输入单引号报错，存在 sql 注入，直接使用 sqlmap 跑，但是没发现信息，查看源码，发现图片的文件名有提示：

```
that-place-where-i-put-that-thing-that-time/565b0f4691fe5
```

发现了目录的名字 that-place-where-i-put-that-thing-that-time 然后再次进行扫描，发现 txt 文件 http://192.168.5.40/that-place-where-i-put-that-thing-that-time/creds.txt 发现：

```
creds.txt -> /etc/julian.txt
```
