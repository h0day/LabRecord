# Djinn

2025.01.03 https://hackmyvm.eu/machines/machine.php?vm=Djinn

## Ip

192.168.5.39

## Scan

```
PORT     STATE    SERVICE
21/tcp   open     ftp
22/tcp   filtered ssh
1337/tcp open     waste
7331/tcp open     swx
```

ftp 可以匿名登陆，将其内容全部下载后查看。

```
nitu:81299

oh and I forgot to tell you I've setup a game for you on port 1337. See if you can reach to the
final level and get the prize.

@nitish81299 I am going on holidays for few days, please take care of all the work.
And don't mess up anything.
```

说 1337 端口上有一个游戏，使用 nc 进行连接，发现是个计算游戏，但是好像没看出什么漏洞。
