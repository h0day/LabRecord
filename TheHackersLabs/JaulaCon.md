# JaulaCon

2025.03.06 https://thehackerslabs.com/jaulacon/

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3333/tcp open  dec-notes
8080/tcp open  http-proxy
```

80 为静态 web，发现 2 个顾客的人名 Jamesh Dame 和 Jumini Kiri ，其他无可用信息。

3333 对应 Node.js，访问后直接跳转到登陆 http://192.168.5.40:3333/login 目前没有登陆用户名和密码，简单测试也不存在 sql 注入。

8080 显示 apache 页面，扫描出了 cgi-bin:

```
gobuster dir -t 100 -k -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt  -u http://192.168.5.40:8080/cgi-bin/ -e
```
