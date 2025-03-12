# Deeper

2025.03.12 https://hackmyvm.eu/machines/machine.php?vm=Deeper

[video](https://www.bilibili.com/video/BV1L8QHYWEsy/?spm_id_from=333.1387.collection.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

web 源码中提示 GO "deeper" 访问 /deeper 目录，又提示 GO evendeeper 再次访问 /deeper/evendeeper 查看源代码发现:

```
<!-- USER .- .-.. .. -.-. . -->
<!-- PASS 53586470624778486230526c5a58426c63673d3d -->
```

user 像是摩尔斯码，进行解码得到 ALICE (要转换成小写的字符) ，对 pass 进行 hex 和 base64 解码，得到字符串 IwillGoDeeper 使用这个凭据 alice:IwillGoDeeper 登陆 ssh 成功。

先拿到了 user flag：

```
alice@deeper:~$ cat user.txt
7e267b737cc121c29b496dc3bcffa5a7
```

发现隐藏文件 bob.txt:

```
alice@deeper:~$ cat .bob.txt
535746745247566c634556756233566e61413d3d
```

同样将十六进制转换成字符串再进行 base64 解码，得到 IamDeepEnough 可能是 bob 的用户密码，切换成功。

发现 /home/bob/root.zip 文件，解压有密码，john 破解 密码为 bob ，解压后得到了 root 的用户密码: root:IhateMyPassword 切换到 root 用户。

拿到最终的 root flag:

```
root@deeper:~# cat root.txt
dbc56c8328ee4d00bbdb658a8fc3e895
```
