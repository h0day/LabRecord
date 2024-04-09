# Lampião: 1

https://www.vulnhub.com/entry/lampiao-1,249/#release

difficulty: Easy

Finish Date：2024-4-7

## IP

192.168.10.151

## Scan

Open Port -> 22,80,1898

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 46b199607d81693cae1fc7ffc366e310 (DSA)
|   2048 f3e888f22dd0b2540b9cad6133595593 (RSA)
|   256 ce632af7536e46e2ae81e3ffb716f452 (ECDSA)
|_  256 c655ca073765e306c1d65b77dc23dfcc (ED25519)
80/tcp   open  http?
| fingerprint-strings:
|   NULL:
|     _____ _ _
|     |_|/ ___ ___ __ _ ___ _ _
|     \x20| __/ (_| __ \x20|_| |_
|     ___/ __| |___/ ___|__,_|___/__, ( )
|     |___/
|     ______ _ _ _
|     ___(_) | | | |
|     \x20/ _` | / _ / _` | | | |/ _` | |
|_    __,_|__,_|_| |_|
1898/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Lampi\xC3\xA3o
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
```

访问 80 端口，无有用信息，继续访问 1898 端口，发现使用的 CMS 为 Drupal，进一步访问 /CHANGELOG.txt，发现使用的版本为：Drupal 7.54, 2017-02-01。

先在 msf 搜索，是否有可以直接利用的 exp，找到了 exploit/unix/webapp/drupal_drupalgeddon2 ，设置相关参数后，进行利用，获得了 shell 权限。

```
meterpreter > getuid
Server username: www-data
shell

升级下tty:
python -c 'import pty; pty.spawn("/bin/bash")'
```

获得 shell 后，cat /etc/passwd 发现一个普通用户：tiago

直接上传 linpeas.sh 进行系统枚举，发现 /var/www/html/sites/default/settings.php 中存在数据库的登陆密码：

```php
databases = array (
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'Virgulino',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

尝试使用发现的密码 tiago:Virgulino 进行 ssh 登陆，发现能通过 ssh 登陆到系统。

另一种思路：

看到 http://192.168.10.151:1898/?q=node/1 是一个文章列表，能看到发表的用户 Submitted by tiago on Thu ，可以利用 cewl 对页面信息进行收集，将收集到的单词作为密码，尝试登陆网站，没有成功。然后尝试用 hydra 对 ssh 进行暴力破解，最终也获得了 tiago 的登陆密码为 Virgulino。

信息收集中，发现内核版本较低，可能存在一系列内核提权漏洞：

```
Linux version 4.4.0-31-generic (buildd@lgw01-01) (gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.3) )
```

尝试使用 dirtycow 进行提权，https://www.exploit-db.com/exploits/40847 按照 exp 中的操作方式进行编译和执行，最终得到 root 权限。

## flag

root@lampiao:~# cat flag.txt
9740616875908d91ddcdaa8aea3af366
