# Binary Trail

2025.02.25 https://thehackerslabs.com/binary-trail/

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
```

只开放 1 个端口，同时看下靶机自带的 pdf 文件，显示了登陆密码： root / toor , 比较简单，寻找系统上的一些特殊文件和日志文件，答案如下。

问题 1: auth_proxy
问题 2: /etc/.shadow_auth
问题 3: touch
问题 4: /var/log/auth.log
问题 5: 600
