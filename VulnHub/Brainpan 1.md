# Brainpan: 1

https://www.vulnhub.com/entry/brainpan-1,51/

difficulty: not know

Finish Date：2024-4-19

## IP

192.168.10.168

## Scan

Open Port -> 9999,10000

```
PORT      STATE SERVICE VERSION
9999/tcp  open  abyss?
| fingerprint-strings:
|   NULL:
|     _| _|
|     _|_|_| _| _|_| _|_|_| _|_|_| _|_|_| _|_|_| _|_|_|
|     _|_| _| _| _| _| _| _| _| _| _| _| _|
|     _|_|_| _| _|_|_| _| _| _| _|_|_| _|_|_| _| _|
|     [________________________ WELCOME TO BRAINPAN _________________________]
|_    ENTER THE PASSWORD
10000/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.3)
|_http-title: Site doesn't have a title (text/html).
```

9999 是一个 tcp 监听端口，nc 连接后，需要我们输入密码，但是暂时不知道。

访问 10000 端口，是一个 web 服务，主页及源代码都没发现什么有用信息。使用目录扫描 ，发现了个/bin 目录，有一个 brainpan.exe，名字和 9999 端口上显示的名字一致。猜测需要我们分析这个 exe 是否有溢出漏洞，然后将 exp 应用到上面的 9999 端口。
