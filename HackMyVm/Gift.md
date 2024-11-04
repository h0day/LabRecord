# Quick

2024-11-04 https://hackmyvm.eu/machines/machine.php?vm=Gift

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先看 80 web 服务，主页没内容，目录扫描后也没发现任何隐藏目录。

直接 rockyou 爆破 22 ssh root 用户的密码为 simple ，ssh 直接登陆获得 flag：

```
gift:~# cat user.txt
HMV665sXzDS
gift:~# cat root.txt
HMVtyr543FG
```
