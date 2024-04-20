# HackLAB: Vulnix

2024-4-20 https://www.vulnhub.com/entry/hacklab-vulnix,48/

difficulty: Low

## IP

192.168.10.170

## Scan

Open Port -> 22,25,79,110,111,143,512,513,514,993,995,2049,41415,43932,52110,54307,59168

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

根据描述，这个靶场存在错误的配置，导致能够获得系统权限。开启的端口比较多，有 smpt 、pop3、 imap 等服务。
