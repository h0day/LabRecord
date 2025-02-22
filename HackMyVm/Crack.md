# Crack

2025.02.22 https://hackmyvm.eu/machines/machine.php?vm=Crack

## Ip

192.168.5.39

## Scan

```
PORT      STATE SERVICE
21/tcp    open  ftp
4200/tcp  open  vrml-multi-use
12359/tcp open  unknown
```

ftp 可以匿名登陆，/upload 目录可以写文件，/upload/crack.py 将其下载，发现就是端口 12359 上运行的 python 文件。

```
no = "NO"
while True:
        try:
                c.send('File to read:'.encode())
                data = c.recv(1024)
                file = (str(data, 'utf-8').strip())
                filename = os.path.basename(file)
                check = "/srv/ftp/upload/"+filename
                if os.path.isfile(check) and os.path.isfile(file):
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

nc 192.168.5.39 12359 输入 crack.py 就可以读取下载的文件，

访问 https://192.168.5.39:4200/ 显示的是一个 shell in a box 但是需要输入用户名和密码。
