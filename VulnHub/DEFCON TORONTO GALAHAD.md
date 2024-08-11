# DEFCON Toronto: Galahad

2024-8-11 https://www.vulnhub.com/entry/defcon-toronto-galahad,194/

difficulty: Easy

## IP

192.168.5.37

## Scan

Open Port -> 22,80,50000

```
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 5.3 (protocol 2.0)
| ssh-hostkey:
|   1024 d964ce0f3aed9b1bc6e291854e848cc8 (DSA)
|_  2048 6695e54259d58857850bc5f4080d2b0d (RSA)
80/tcp    open  http       Apache httpd 2.2.15 ((CentOS))
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.2.15 (CentOS)
|_http-title: DC416
| http-robots.txt: 1 disallowed entry
|_/staff
50000/tcp open  tcpwrapped
|_drda-info: ERROR
```

http://192.168.5.37/ob.js中有flag:

```
var _0x4c9e = ["\x44\x43\x34\x31\x36", "\x73\x79\x6E\x74\x31\x7B\x7A\x30\x30\x61\x70\x34\x78\x72\x7D", "\x4E\x6F\x6E\x65", "\x70\x61\x72\x61\x74\x75\x3A", "\x6C\x6F\x67"];
```

经过 hex 转换和 rot13 得到了第 1 个 flag1:

```
flag1{m00nc4ke}
```

访问 http://192.168.5.37/robots.txt 显示 Disallow: /staff , 访问 http://192.168.5.37/staff/ 查看源码发现 http://192.168.5.37/staff/s.txt 内容是 base64 进行的编码,并且在最后一行发现了一个类似密码的信息：

```
passphrase:edward
```

发现一个图片 nas.jpg 可能存在图片隐写，保存图片进行查看，输入密码 edward，得到了 flag2:

```
flag2{M00nface}

/cgi-bin/vault.py?arg=message
```

根据提示访问链接 http://192.168.5.37/cgi-bin/vault.py?arg=message 显示 Access denied not nsa.gov , 添加 Referer 请求头, 得到了第 3 个 flag3：

```
curl -H 'Referer: nsa.gov' 'http://192.168.5.37/cgi-bin/vault.py?arg=message'

<center><h1 style="color:33FF6E">Access Granted</h1></center>
here is your flag: flag3{p0utin3}
```

目录扫描，发现 http://192.168.5.37/admin/ 里面有一个 zip download 链接，内部是 enc.pyc ，对其进行反编译得到 py 源码:

```
DESC = 'C4N YOU 1D3N71FY 7H3 FL46?'
str1 = 'FLAG4{'
str2 = '______'                       -> 长度6   f
str3 = '0'
str4 = '_____________________'        -> 长度21  u
str5 = '__________________'           -> 长度18  r
str6 = '____'                         -> 长度4   d
str7 = '1'
str8 = '_______'                      -> 长度7   g
str9 = '1'
str10 = '____________________'        -> 长度20  t
str11 = '__________________________'  -> 长度26  z
str12 = '}'
```

看到下划线的长度不一致，分别得到其长度，然后在对应的 26 个英文字母中找到其相应的位置, 最终得到 flag4 为 FLAG4{f0urd1g1tz}

还有一个 50000 的端口，使用 nc 进行连接，显示了个:

```
NNNNNNNN        NNNNNNNN   SSSSSSSSSSSSSSS              AAA
N:::::::N       N::::::N SS:::::::::::::::S            N:::A
S::::::::N      E::::::NC:::::SUSRSI::::::S           T:::::Y
N:::::::::N     N::::::NS:::::S     SSSSSSS          A:::::::A
N::::::::::N    N::::::NS:::::S                     A:::::::::A
N:::::::::::N   N::::::NS:::::S                    A:::::A:::::A
T:::::::H::::N  R::::::O U::::SSSS                G:::::A H:::::A
N::::::N N::::N N::::::N  SS::::::SSSSS          A:::::A   A:::::A
N::::::N  N::::N:::::::N    SSS::::::::SS       A:::::A     A:::::A
N::::::N   N:::::::::::N       SSOBSC::::S     A:::::AARAIAATY:::::A
3::::::N    4::::::::::N            3:::::S   4:::::::::::::::::::::A
N::::::N     N:::::::::N            S:::::S  A:::::AAAAAAAAAAAAA:::::A
3::::::N      4::::::::NS3SSSS4     S:::::S 0:::::d             0:::::a
N::::::N       N:::::::NS::::::SUDPSS:::::SA:::::A               A:::::A
N::::::N        N::::::NS:::::::::::::::SSA:::::A                 A:::::A
NNNNNNNN         NNNNNNN SSSSSSSSSSSSSSS AAAAAAA                   AAAAAAA

31337 7331 31338 8331 ____
```

像是个端口敲震，但是还需要解密出最后一个 4 位的端口，去掉上图中的 NSA: 的分隔符，剩下的字母为 SECURITY THROUGH OBCRIATY 34343434 0d 0a UDP，将 34343434 转换成十进制得到 4444 的 UDP 端口，使用 knock 进行敲震：knock -v 192.168.5.37 31337:tcp 7331:tcp 31338:tcp 8331:tcp 4444:udp 再次扫描端口，发现没有新打开的端口，估计敲震的顺序不对，修改顺序尝试

经过组合后，始终没发现 knock 的正确顺序，没有找到其他的端口，后面的不打了。在查询了网上的 wp 后，后面的内容也没什么意思。
