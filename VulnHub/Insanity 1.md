# Insanity: 1

2024-5-x https://www.vulnhub.com/entry/insanity-1,536/

difficulty: Intermediate

## IP

192.168.5.28

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: ERROR
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.5.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 85464106da830401b0e41f9b7e8b319f (RSA)
|   256 e49cb1f244f1f04bc38093a95d9698d3 (ECDSA)
|_  256 65cfb4afad8656efae8bbff2f0d9be10 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/7.2.33)
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.2.33
|_http-title: Insanity - UK and European Servers
```

21 ftp 可以匿名登陆，登陆后，有一个 pub 文件夹，里面什么都没有，同时匿名用户不能上传文件。

直接看 80 web 服务，http://192.168.5.28/ 没发现其他连接，使用 gobuster 进行扫描：

```
gobuster -t 64 dir -u http://192.168.5.28/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.5.28/img                  (Status: 301) [Size: 232] [--> http://192.168.5.28/img/]
http://192.168.5.28/data                 (Status: 301) [Size: 233] [--> http://192.168.5.28/data/]
http://192.168.5.28/news                 (Status: 301) [Size: 233] [--> http://192.168.5.28/news/]
http://192.168.5.28/index.php            (Status: 200) [Size: 31]
http://192.168.5.28/index.html           (Status: 200) [Size: 22263]
http://192.168.5.28/css                  (Status: 301) [Size: 232] [--> http://192.168.5.28/css/]
http://192.168.5.28/js                   (Status: 301) [Size: 231] [--> http://192.168.5.28/js/]
http://192.168.5.28/webmail              (Status: 301) [Size: 236] [--> http://192.168.5.28/webmail/]
http://192.168.5.28/fonts                (Status: 301) [Size: 234] [--> http://192.168.5.28/fonts/]
http://192.168.5.28/monitoring           (Status: 301) [Size: 239] [--> http://192.168.5.28/monitoring/]
http://192.168.5.28/licence              (Status: 200) [Size: 57]
http://192.168.5.28/phpmyadmin           (Status: 301) [Size: 239] [--> http://192.168.5.28/phpmyadmin/]
http://192.168.5.28/phpinfo.php          (Status: 200) [Size: 85264]
```

http://192.168.5.28/webmail/src/login.php 发现一个 SquirrelMail version 1.4.22 服务。

http://192.168.5.28/monitoring/login.php 这里发现了一个登陆页面。

http://192.168.5.28/phpmyadmin/ 发现 phpmyadmin 登陆页面。

SquirrelMail version 1.4.22 存在 RCE 漏洞，但是需要一个可以登陆的用户凭据：

```
SquirrelMail < 1.4.22 - Remote Code Execution  | linux/remote/41910.sh
```
