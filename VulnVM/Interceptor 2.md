# Interceptor 2

2025.03.26 https://www.vulnvm.com/interceptor-2

[video]()

## Ip

192.168.5.40

## Scan

```
PORT    STATE SERVICE
21/tcp  open  ftp
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

smb 上枚举出一个用户 smbuser ， 很长时间才爆破出它的密码 `_z<+0e05AXKv` , 使用 smbclient 进行登陆，/srv/smbshare 目录发现 watch_run.sh 脚本，在执行文件监控:

```
#!/bin/bash
inotifywait -m -e modify,create,move,close_write /srv/smbshare | while read path action file; do
    if [[ "$file" == *.sh ]]; then
        chmod +x "/srv/smbshare/$file"
        sudo -u www-data /bin/bash "/srv/smbshare/$file" &
    fi
done
```

这时直接上传一个 reverse.sh，上传之后就会触发 inotifywait，进而以 www-data 用户身份执行反弹 shell，系统枚举 sudo -l 显示 (ALL) NOPASSWD: /srv/smbshare/run.sh ， 在通过 smb 上传一个 run.sh 反弹 shell 脚本，就拿到了 root 权限。

最终读取到了 2 个 flag：

```
root@debian:/home/vincent# cat user.txt
647cb9f7681a16e4d624292af30ac0cf
```

```
root@debian:~# cat root.txt
ece9596c8b1cff09b05ececadc0bc5b4
```
