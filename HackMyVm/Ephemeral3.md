# Ephemeral3

2025.02.10 https://hackmyvm.eu/machines/machine.php?vm=Ephemeral3

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web，主页是 apache 默认页面，gobuster 进行扫描，发现: http://192.168.5.40/note.txt

```
Hey! I just generated your keys with OpenSSL. You should be able to use your private key now!
If you have any questions just email me at henry@ephemeral.com
```

但是目前没什么用。

发现另外一个目录: http://192.168.5.40/agency
