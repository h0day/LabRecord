# Hundred

2024-11-22 https://hackmyvm.eu/machines/machine.php?vm=Hundred

## IP

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

在访问 80 查看源代码发现：

```
key: h4ckb1tu5.enc;

<!-- l4nr3n, nice dir.-->
```

看着 h4ckb1tu5.enc 像是一个域名或者是一个文件，先添加到 hosts 文件中，进行访问，根主页内容一样，但是访问 l4nr3n 这个目录还是 404。在访问下看看 h4ckb1tu5.enc 是不是文件，访问成功，是个文件：

```
file h4ckb1tu5.enc
h4ckb1tu5.enc: data
```

enc 文件是加密后的文件，可以是用 openssl 进行解密，

ftp 允许匿名登陆，先看 ftp，里面有如下文件：

```
-rwxrwxrwx    1 0        0             435 Aug 02  2021 id_rsa
-rwxrwxrwx    1 1000     1000         1679 Aug 02  2021 id_rsa.pem
-rwxrwxrwx    1 1000     1000          451 Aug 02  2021 id_rsa.pub
-rwxrwxrwx    1 0        0             187 Aug 02  2021 users.txt
```

userx.txt 中发现类似用户的字典：

```
noname
roelvb
ch4rm
marcioapm
isen
sys7em
chicko
tasiyanci
luken
alienum
linked
tatayoyo
0xr0n1n
exploiter
kanek180
cromiphi
softyhack
b4el7d
val1d
```

id_rsa.pem 中发现了私钥，使用此私钥对上面的用户字典尝试 ssh 密钥登陆，
