# sunset: noontide

2024-5-19 https://www.vulnhub.com/entry/sunset-noontide,531/

difficulty: easy

## IP

192.168.5.21

## Scan

Open Port -> 6667,6697,8067

```
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
6697/tcp open  irc     UnrealIRCd (Admin email example@example.com)
8067/tcp open  irc     UnrealIRCd
```

通过在 msf 中搜索，找到了利用的 exp unix/irc/unreal_ircd_3281_backdoor，同时注意，目标程序的 payload 需要使用 perl 进行反弹 cmd/unix/reverse_perl，而其他的 cmd/unix/reverse 不行。设置好 rhost 后，执行就会得到反弹的 shell。

没有 sudo 程序，suid 也没有发现可利用的程序

尝试使用 su root 输入密码：root，神奇的变成了 root 权限，根据靶场描述类似，狠简单不要想复杂，是个弱密码：

```
server@noontide:~$ cat local.txt
c53c08b5bf2b0801c5d0c24149826a6e
server@noontide:~$ su
Password:
root@noontide:/home/server# cd /root
root@noontide:~# cat proof.txt
ab28c8ca8da1b9ffc2d702ac54221105
```
