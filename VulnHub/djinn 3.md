# djinn: 3

2024-5-x https://www.vulnhub.com/entry/djinn-3,492/

difficulty: Intermediate

## IP

192.168.5.29

## Scan

Open Port -> 22,80,5000,31337

```
PORT      STATE SERVICE VERSION
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e64423acb2d982e79058155e4023ed65 (RSA)
|   256 ae04856ecb104f554aad969ef2ce184f (ECDSA)
|_  256 f708561997b5031018667e7d2e0a4742 (ED25519)
80/tcp    open  http    lighttpd 1.4.45
|_http-server-header: lighttpd/1.4.45
|_http-title: Custom-ers
5000/tcp  open  http    Werkzeug httpd 1.0.1 (Python 3.6.9)
|_http-server-header: Werkzeug/1.0.1 Python/3.6.9
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
31337/tcp open  Elite?
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, NULL:
|     username>
|   GenericLines, GetRequest, HTTPOptions, RTSPRequest, SIPOptions:
|     username> password> authentication failed
|   Help:
|     username> password>
|   RPCCheck:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|     UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80 in position 0: invalid start byte
|   SSLSessionReq:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|     UnicodeDecodeError: 'utf-8' codec can't decode byte 0xd7 in position 13: invalid continuation byte
|   TerminalServerCookie:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|_    UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe0 in position 5: invalid continuation byte
```

先看 80 端口的 web 服务，gobuster：

```
gobuster -t 64 dir -u http://192.168.5.29/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e
```

什么都没扫出来，80 是个 html 静态页面。

看看 5000 端口的服务，是一个 python 搭建的服务 Werkzeug/1.0.1，显示了一个数据列表，显示了一些系统修复的信息。打开那几个连接，得到了一些提示信息：

```
可能存在用户名 jason、david、freddy、umang、jack

RLUI 团队

使用的数据库 Postgres

默认用户 guest

可能存在弱密码

用户可通过密码以及通过 API 密钥或令牌进行授权
```

http://192.168.5.29:5000/?id=8345 遍历一下这个 id 的值，看看会不会有其他重要信息泄露，从 1-20000 没发现什么信息，就只有那 6 个 id 对应的信息。

http://192.168.5.29:5000/?id=8345 发现这个 id 的值会显示在页面中，有没有可能存在模板注入，尝试寻找。

31337 是一个 tcp 服务，应该就是 5000 端口中说的票务系统，可以用 nc 进行连接，但是需要用户名和密码。经过探测使用的是 python 搭建的。

前面发现了肯能存在的几个用户名: jason、david、freddy、umang、jack
