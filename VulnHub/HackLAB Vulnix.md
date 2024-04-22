# HackLAB: Vulnix

2024-4-20 https://www.vulnhub.com/entry/hacklab-vulnix,48/

difficulty: Low

## IP

192.168.10.170

## Scan

Open Port -> 22,25,79,110,111,143,512,513,514,993,995,2049,35402,41639,50355,56639,57172

```
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 10cd9ea0e4e030243ebd675f754a33bf (DSA)
|   2048 bcf924072fcb76800d27a648520a243a (RSA)
|_  256 4dbb4ac118e8dad1826f58529cee345f (ECDSA)
25/tcp    open  smtp     Postfix smtpd
|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
79/tcp    open  finger   Linux fingerd
|_finger: No one logged on.\x0D
110/tcp   open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2024-04-20T09:35:59+00:00; +7h59m59s from scanner time.
|_pop3-capabilities: TOP RESP-CODES STLS SASL CAPA UIDL PIPELINING
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      37429/tcp6  mountd
|   100005  1,2,3      39101/udp6  mountd
|   100005  1,2,3      41415/tcp   mountd
|   100005  1,2,3      49944/udp   mountd
|   100021  1,3,4      44178/udp   nlockmgr
|   100021  1,3,4      52156/udp6  nlockmgr
|   100021  1,3,4      54307/tcp   nlockmgr
|   100021  1,3,4      60150/tcp6  nlockmgr
|   100024  1          33639/tcp6  status
|   100024  1          50864/udp   status
|   100024  1          52110/tcp   status
|   100024  1          52865/udp6  status
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
143/tcp   open  imap     Dovecot imapd
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2024-04-20T09:35:59+00:00; +7h59m59s from scanner time.
|_imap-capabilities: more SASL-IR listed OK have capabilities STARTTLS Pre-login post-login LITERAL+ IDLE LOGIN-REFERRALS ENABLE IMAP4rev1 LOGINDISABLEDA0001 ID
512/tcp   open  exec     netkit-rsh rexecd
513/tcp   open  login?
514/tcp   open  shell    Netkit rshd
993/tcp   open  ssl/imap Dovecot imapd
|_ssl-date: 2024-04-20T09:35:39+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_imap-capabilities: ID more OK have capabilities listed Pre-login post-login SASL-IR IDLE LITERAL+ ENABLE LOGIN-REFERRALS IMAP4rev1 AUTH=PLAINA0001
995/tcp   open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: TOP RESP-CODES USER SASL(PLAIN) CAPA UIDL PIPELINING
|_ssl-date: 2024-04-20T09:35:39+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
2049/tcp  open  nfs_acl  2-3 (RPC #100227)
41415/tcp open  mountd   1-3 (RPC #100005)
43932/tcp open  mountd   1-3 (RPC #100005)
52110/tcp open  status   1 (RPC #100024)
54307/tcp open  nlockmgr 1-4 (RPC #100021)
59168/tcp open  mountd   1-3 (RPC #100005)
```

根据描述，这个靶场存在错误的配置，导致能够获得系统权限。

开启的端口比较多，有 smpt 、pop3、 imap、NFS 等服务。

对于邮件服务，我们手里没有可用的用户名和密码，只能尝试去枚举下可能存在的用户名：

```
smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t 192.168.10.170

或者使用 msf 中的 scanner/smtp/smtp_enum
```

得到了如下的非系统自带用户名：

```
user
```

再看看 NFS 2049 上有什么信息：

```
showmount -e 192.168.10.170
Export list for 192.168.10.170:
/home/vulnix *
```

看到有一个 /home/vulnix 目录，从这个上面我们能获得一个用户名：vulnix，让我们挂载到 kali 上看看里面有什么信息：

```
mkdir -p /tmp/mount
sudo mount -o rw,vers=3 192.168.10.170:/home/vulnix /tmp/mount
```

无法 cd 到 /tmp/mount ，ls -al /tmp/mount 发现所属用户的 ID 是 2008，需要我们在 kali 上床架一个 uid 为 2008 的账户才能进入此目录 sudo useradd -m -u 2008 vulnix，创建账户后，使用切换到该账户，然后进入 /tmp/mount 目录，让我们看看我们是否有写权限，尝试创建一个新的文件，发现有 w 权限。由于看不到目标主机的 NFS 的配置文件，可能这个目录上存在 no_root_squash 配置，导致我们能留下 suid 文件，为后续提升到 root 权限做准备。使用 root 用户创建一个文件，复制到此目录中时，发现文件的所属用户变成了 nobody，看样子 NFS 的配置中没有设置 no_root_squash ，这个利用暂时没什么用。

端口 79 对应的服务为 finger ，可以使用它获取到一些系统的用户信息，使用 msf 中提供的相关扫描模块：

```
use auxiliary/scanner/finger/finger_users
set rhsots 192.168.10.170
run
```

枚举出系统有如下用户列表：

```
backup, bin, daemon, dovecot, dovenull, games, gnats, irc, landscape, libuuid, list, lp, mail, man, messagebus, news, nobody, postfix, proxy, root, sshd, sync, sys, syslog, user, uucp, whoopsie, www-data
```

如果有可能话的话，有了用户列表，我们就能利用 ssh 进行密码爆破。

在尝试了其他端口后，没有发现有用信息，我们现在收集到的就是一堆用户名，那么只能尝试 ssh 进行爆破了，把 finger 中搜集的用户名进行过滤，去掉系统保留的用户名，留下：

```
user
```

使用 hydra 爆破 ssh 22 端口，只找到了 user 的密码：letmein

使用得到的用户凭证，ssh 登陆，直接使用 linpeas 进行系统枚举，找到提升到 root 的攻击面。

linpeas 中只发现了内核版本 3.2.0-29-generic-pae 可能存在漏洞，未发现其他攻击面。我们注意到前面的枚举中，/home 目录下有一个 vulnix 用户，这个用户上可能存在攻击面，我们是否能登陆到这个用户上？可以利用前面的 NFS 将生成的 ssh 公钥复制到它的目录中/tmp/mount/.ssh/authorized_keys，然后远程登陆。

远程登陆后，查看相关权限：

```
sudo -l
ser vulnix may run the following commands on this host:
    (root) sudoedit /etc/exports, (root) NOPASSWD: sudoedit /etc/exports
```

我们可以使用 sudoedit /etc/exports 将 /home/vulnix \*(rw,root_squash) 中的 root_squash 改成 no_root_squash，这样我们就能在 kali 上以 root 身份，上传 suid 文件，并且进行执行。或者将/home/vulnix 的目录改成/root，我们就能够直接读取/root 目录的文件信息。

另外，也可以用 dircow 40839.c 进行提权到 root，将在 32 位环境下编译好的 exp 上传到目标机器，执行后可进行提权。
