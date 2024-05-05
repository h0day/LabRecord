# SpyderSec: Challenge

2024-5-5 https://www.vulnhub.com/entry/spydersec-challenge,128/

difficulty: Intermediate

## IP

192.168.5.26

## Scan

Open Port -> 22,80

```
PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd
|_http-title: SpyderSec | Challenge
|_http-server-header: Apache
```

22 ssh close。

80 端口是 web 服务，使用目录扫描，没发现什么有用信息，只扫描出了几个图片信息：

```
200      GET      396l     2144w   152060c http://192.168.5.26/Challenge.png
200      GET        1l      134w     1918c http://192.168.5.26/favicon.ico
200      GET      142l      700w    46821c http://192.168.5.26/SpyderSecLogo200.png
200      GET       32l      822w     8883c http://192.168.5.26/
200      GET       32l      822w     8883c http://192.168.5.26/index.php
301      GET        9l       27w      292c http://192.168.5.26/v => http://192.168.5.26/v/
```

http://192.168.5.26/v/ 访问不了 403。

主页显示了找到 flag 的相关提示，源代码有一段 js 自执行代码：

```
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('7:0:1:2:8:6:3:5:4:0:a:1:2:d:c:b:f:3:9:e',16,16,'6c|65|72|27|75|6d|28|61|74|29|64|62|66|2e|3b|69'.split('|'),0,{}))
```

让我们在浏览器控制台中执行这段 js 代码，看能得到什么内容：

```
function ab(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}

ab('7:0:1:2:8:6:3:5:4:0:a:1:2:d:c:b:f:3:9:e',16,16,'6c|65|72|27|75|6d|28|61|74|29|64|62|66|2e|3b|69'.split('|'),0,{});
```

得到结果：

```
61:6c:65:72:74:28:27:6d:75:6c:64:65:72:2e:66:62:69:27:29:3b
```

使用 cyberchef 进行解码：

```
alert('mulder.fbi');
```

像是一个文件名，先放这里。

在看看请求链接：

```
curl -I http://192.168.5.26/
HTTP/1.1 200 OK
Date: Sun, 05 May 2024 11:37:50 GMT
Server: Apache
Set-Cookie: URI=%2Fv%2F81JHPbvyEQ8729161jd6aKQ0N4%2F; expires=Mon, 06-May-2024 11:37:50 GMT; path=/; httponly
Connection: close
Content-Type: text/html; charset=UTF-8
```

URI 中的信息有些特殊，是一个 url 链接的后半部分，让我们拼接上继续访问 http://192.168.5.26/v/81JHPbvyEQ8729161jd6aKQ0N4/ ，但是还是 403，把上面得到的疑似文件名 mulder.fbi 拼接到后面：http://192.168.5.26/v/81JHPbvyEQ8729161jd6aKQ0N4/mulder.fbi ，得到了一个下载文件，是一个 MP4 视频。

视频中没看出什么内容，考虑视频中是否有信息隐写，寻找工具进行分析。

尝试使用 VeraCrypt 或 TrueCrypt 对视频进行解密，但是需要一个输入一个密码，根据 vulnhub 描述中的信息，我们需要找到另外一个文件，下面继续寻找。

下载安装 VeraCrypt：

```
https://www.veracrypt.fr/en/Downloads.html
```

在看看 Challenge.png 和 SpyderSecLogo200.png ，是否有图片隐写。

strings Challenge.png 查看可显示的字符串内容：

```
iTXtComment
35:31:3a:35:33:3a:34:36:3a:35:37:3a:36:34:3a:35:38:3a:33:35:3a:37:31:3a:36:34:3a:34:35:3a:36:37:3a:36:61:3a:34:65:3a:37:61:3a:34:39:3a:33:35:3a:36:33:3a:33:30:3a:37:38:3a:34:32:3a:34:66:3a:33:32:3a:36:37:3a:33:30:3a:34:61:3a:35:31:3a:33:64:3a:33:64
```

使用 CyberChef，使用 Magic 进行探测性解码，最后得到了一串 base64 代码：

```
QSFWdX5qdEgjNzI5c0xBO2g0JQ==
```

解码得到：

```
A!Vu~jtH#729sLA;h4%
```

这里应该就是上面解密视频需要的密钥，输入后，得到的隐藏在视频中的信息：

```
----------------
Congratulations!

You are a winner.

Please leave some feedback on your thoughts regarding this challenge?Was it fun? Was it hard enough or too easy? What did you like or dislike, what could be done better?

https://www.spydersec.com/feedback
---------------
```
