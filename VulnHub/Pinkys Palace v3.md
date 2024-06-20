# Pinky's Palace: v3

2024-6-20 https://www.vulnhub.com/entry/pinkys-palace-v3,237/

difficulty: begginer-intermediate

## IP

192.168.5.37

## Scan

Open Port -> 21,5555,8000

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
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
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             173 May 14  2018 WELCOME
5555/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey:
|   2048 80526ebdb0c4be0af21d3bacb8474fee (RSA)
|   256 ebc876a4cf376f0d5ff548af5c2992d9 (ECDSA)
|_  256 482b84023e877b2af39111310f9811c7 (ED25519)
8000/tcp open  http    nginx 1.10.3
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-title: PinkDrup
|_http-server-header: nginx/1.10.3
```

先看 ftp 可匿名登陆，WELCOME 提示信息:

```
Welcome to Pinky's Palace V3

Good Luck ;}

I encourage you to be creative, try and stay away from metasploit and pre-made tools.
You will learn much more this way!

~Pinky
```

另外发现了一个 ... 的隐藏文件夹，里面有一个 .bak 文件夹，firewall.sh 下载下来看看什么内容:

```
iptables -A OUTPUT -o eth0 -p tcp --tcp-flags ALL SYN -m state --state NEW -j DROP
ip6tables -A OUTPUT -o eth0 -p tcp --tcp-flags ALL SYN -m state --state NEW -j DROP
```

发现防火墙对出战的链接有限制，可能影响后续的反弹 shell.

转战 8000 端口，CMS 为 Drupal，http://192.168.5.37:8000/CHANGELOG.txt 发现版本为 7.57 尝试使用了 msf 中的 drupal_drupalgeddon2 进行利用, 反弹没有成功，尝试用 php/bind_php， 重新执行后得到了 web shell。

进入 shell 后，先找到 CMS 的数据库配置文件 ./sites/default/settings.php :

```
$databases = array (
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'drupal',
      'username' => 'dpink',
      'password' => 'drupink',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

发现几个可用用户:

```
cat /etc/passwd | grep '/bin/bash'
root:x:0:0:root:/root:/bin/bash
pinky:x:1000:1000:pinky,,,:/home/pinky:/bin/bash
pinksec:x:1001:1001::/home/pinksec:/bin/bash
pinksecmanagement:x:1002:1002::/home/pinksecmanagement:/bin/bash
```

尝试用得到的数据库密码进行喷射，
