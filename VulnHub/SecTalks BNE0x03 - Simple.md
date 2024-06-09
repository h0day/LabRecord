# SecTalks: BNE0x03 - Simple

2024-6-9 https://www.vulnhub.com/entry/sectalks-bne0x03-simple,141/

difficulty: easy

## IP

192.168.5.31

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Please Login / CuteNews
|_http-server-header: Apache/2.4.7 (Ubuntu)
```

访问 80 web 首页，http://192.168.5.31/ 显示的 CMS 是 CuteNews v.2.0.3 经过 searchsploit 发现对应这个版本有文件上传漏洞 CuteNews 2.0.3 - Arbitrary File Upload | php/webapps/37474.txt ，但是需要用户登陆凭据。

使用 gobuster 进行扫描吧，看看能不能找到用户凭据的线索。

发现了一个链接 http://192.168.5.31/cdata/users.txt 显示 nug1x7:1 使用登陆提示密码错误，nug1x7 用户名应该是正确的，使用 hydra+rockyou 进行爆破，没找到密码。

看到主页上有注册的功能，可以先注册个用户，这时就有了用户凭据，就可以使用上面的文件上传漏洞上传 web shell 了。

按照 37474.txt 中的步骤进行操作，上传 Avatar ，使用 bp 进行拦截改包，kali 上监听 8888，curl 访问 http://192.168.5.31/uploads/avatar_test_8888.php，这时得到了反弹的 web shell，先升级下 tty。

发现 /home 目录中有 bull，但是该文件夹中没有提示信息。

linpeas 也没发现有用信息，尝试内核提权吧：

```
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation       | linux/local/37292.c
```

在目标机器上编译 37292.c 并执行，得到了 root 权限：

```
www-data@simple:/tmp$ gcc 37292.ofs.c -o ofs
www-data@simple:/tmp$ ./ofs
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
# cat /root/flag.txt
U wyn teh Interwebs!!1eleven11!!1!
Hack the planet!
```
