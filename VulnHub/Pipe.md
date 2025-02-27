# /dev/random: Pipe

2025.02.27 https://www.vulnhub.com/entry/devrandom-pipe,124/

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
57505/tcp open  unknown
```

访问 http://192.168.5.40/ 是 http-basic 认证，目前没有用户名和密码。

目录扫描 http://192.168.5.40/images/pipe.jpg 下载图片查看隐写，没有隐写。

发现另外一个目录 http://192.168.5.40/scriptz/ 里面有 log.php.BAK 和 php.js ， 下载 BAK 文件，里面有 php 代码，后面可能使用 php 反序列化。

需要找到用户名和密码通过签名的 http 认证才能继续进行。
