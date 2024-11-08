# Gigachad

2024-11-08 https://hackmyvm.eu/machines/machine.php?vm=Gigachad

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 允许匿名登陆，看看有什么信息，下载 chadinfo 文件，是个 zip 文件进行解压，内容为：

```
why yes,
#######################
username is chad
???????????????????????
password?
!!!!!!!!!!!!!!!!!!!!!!!
go to /drippinchad.png
```

发现用户名 chad 密码提示去 /drippinchad.png

通过 80 web 服务访问首页，在 html 源代码的底部发现提示：

```
A7F9B77C16A3AA80DAA4E378659226F628326A95
D82D10564866FD9B201941BCC6C94022196F8EE8 -->
```

访问图片 http://192.168.5.40/drippinchad.png 没有图片隐写，尝试去找到图片中的地点名称，可能是密码，是少女塔：MaidensTower

尝试 ssh 登陆，最终密码为: maidenstower

得到了 userflag ：

```
chad@gigachad:~$ cat user.txt
0FAD8F4B099A26E004376EAB42B6A56A
```

发现 suid 程序 /usr/lib/s-nail/s-nail-privsep 查看其是否有可以利用的地方，发现 exploit db 中的 searchsploit s-nail 将 47172.sh 上传的目标机器上，执行得到了 root 权限：

```
cat root.txt
832B123648707C6CD022DD9009AEF2FD
```
