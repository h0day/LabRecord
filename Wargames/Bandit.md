# Bandit 2024.06.18

## Level 0

ssh -C -p 2220 bandit0@bandit.labs.overthewire.org

passwd:bandit0

## Level 1

```
cat readme

The password you are looking for is: ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If
```

ssh -C -p 2220 bandit1@bandit.labs.overthewire.org

passwd:ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If

## Level 2

```
cat ./-

263JGJPfgU6LtdEvgfWU1XP5yac29mFx
```

ssh -C -p 2220 bandit2@bandit.labs.overthewire.org

passwd:263JGJPfgU6LtdEvgfWU1XP5yac29mFx

## Level 3

```
cat ./spaces\ in\ this\ filename
cat 'spaces in this filename'

MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx
```

ssh -C -p 2220 bandit3@bandit.labs.overthewire.org

passwd:MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

## Level 4

```
cd  inhere/
cat '...Hiding-From-You'

2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
```

ssh -C -p 2220 bandit4@bandit.labs.overthewire.org

passwd:2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

## Level 5

```
cd inhere/
file ./*
./-file07: ASCII text

cat ./-file07
4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
```

ssh -C -p 2220 bandit5@bandit.labs.overthewire.org

passwd:4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw

## Level 6

```
cd inhere/
find . -type f -size 1033c -not -executable 2>/dev/null

cat ./maybehere07/.file2
HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
```

ssh -C -p 2220 bandit6@bandit.labs.overthewire.org

passwd:HWasnPhtq9AVKe0dmk45nxy20cvUa6EG

## Level 7

```
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password

at /var/lib/dpkg/info/bandit7.password
morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj
```

ssh -C -p 2220 bandit7@bandit.labs.overthewire.org

passwd:morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj

## Level 8

```
grep -i 'millionth' data.txt
millionth	dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
```

ssh -C -p 2220 bandit8@bandit.labs.overthewire.org

passwd:dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc

## Level 9

```
cat data.txt |sort |uniq -u
4CKMh1JI91bUIZZPXDqGanal4xvAg0JM
```

ssh -C -p 2220 bandit9@bandit.labs.overthewire.org

passwd:4CKMh1JI91bUIZZPXDqGanal4xvAg0JM

## Level 10

```
strings data.txt | grep -i '=*'
========== FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey
```

ssh -C -p 2220 bandit10@bandit.labs.overthewire.org

passwd:FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey

## Level 11

```
cat data.txt |base64 -d
The password is dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr
```

ssh -C -p 2220 bandit11@bandit.labs.overthewire.org

passwd:dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr

## Level 12

```
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
The password is 7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4
```

ssh -C -p 2220 bandit12@bandit.labs.overthewire.org

passwd:7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4

## Level 13

```
cd /tmp
mkdir www
cd www
cp ~/data.txt .
xxd -r data.txt > data.bin
file data.bin    # gzip compressed data
mv data.bin data.gz
gzip -d data.gz
file data        # bzip2
mv data data.bz2
bzip2 -d data.bz2
file data       # gzip compressed data
mv data data.gz
gzip -d data.gz
file data       # POSIX tar archive
mv data data.tar
tar xvf data.tar
file data5.bin  # POSIX tar archive
mv data5.bin data5.tar
tar xvf data5.tar
file data6.bin  # bzip2 compressed data
mv data6.bin data6.bz2
bzip2 -d data6.bz2
file data6      # POSIX tar archive
mv data6 data6.tar
tar xvf data6.tar
file data8.bin  # gzip compressed data
mv data8.bin data8.gz
gzip -d data8.gz
cat data8
The password is FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn
```

ssh -C -p 2220 bandit13@bandit.labs.overthewire.org

passwd:FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn

## Level 14

```
ssh -p 2220 -i sshkey.private bandit14@bandit.labs.overthewire.org

cat /etc/bandit_pass/bandit14
MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS
```

ssh -C -p 2220 bandit14@bandit.labs.overthewire.org

passwd:MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS

## Level 15

```
cat /etc/bandit_pass/bandit14 | nc localhost 30000
Correct!
8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo
```

ssh -C -p 2220 bandit15@bandit.labs.overthewire.org

passwd:8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo

## Level 16

```
echo '8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo' | ncat --ssl localhost 30001
Correct!
kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
```

ssh -C -p 2220 bandit16@bandit.labs.overthewire.org

passwd:kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx

## Level 17

```
nmap -sT --min-rate 10000 -p31000-32000 localhost
nmap -sC -sV -p31046,31518,31691,31790,31960 localhost

PORT      STATE SERVICE     VERSION
31046/tcp open  echo
31518/tcp open  ssl/echo
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=SnakeOil
| Not valid before: 2024-06-10T03:59:50
|_Not valid after:  2034-06-08T03:59:50
31691/tcp open  echo
31790/tcp open  ssl/unknown
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=SnakeOil
| Not valid before: 2024-06-10T03:59:50
|_Not valid after:  2034-06-08T03:59:50
| fingerprint-strings:
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, LPDString, RTSPRequest, SIPOptions:
|_    Wrong! Please enter the correct current password.
31960/tcp open  echo

IPS=(31046 31518 31691 31790 31960);for ip in ${IPS[@]}; do echo -n $ip ' -> '; echo 'kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx' | ncat --ssl localhost $ip; done

31046  -> Ncat: Input/output error.
31518  -> kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
31691  -> Ncat: Input/output error.
31790  -> Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----

31960  -> Ncat: Input/output error.
```

ssh -i id_rsa -C -p 2220 bandit17@bandit.labs.overthewire.org

## Level 18

```
diff passwords.old passwords.new
42c42
< znK19XRJuZTd8BvCEVW4NQjtNJbdFLNC
---
> x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO
```

ssh -C -p 2220 bandit18@bandit.labs.overthewire.org

passwd:x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO

```
Byebye !
Connection to bandit.labs.overthewire.org closed.
```

## Level 19

ssh -C -p 2220 bandit18@bandit.labs.overthewire.org 'bash --norc --noprofile'

passwd:x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO

```
cat readme
cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8
```

ssh -C -p 2220 bandit19@bandit.labs.overthewire.org

passwd:cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8

## Level 20

```
./bandit20-do cat /etc/bandit_pass/bandit20
0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
```

ssh -C -p 2220 bandit20@bandit.labs.overthewire.org

passwd:0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO

## Level 21

```
echo 0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO | nc -lvnp 9999 &

bandit20@bandit:~$ ./suconnect 9999
Connection received on 127.0.0.1 48838
Read: 0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
Password matches, sending next password
EeoULMCra2q0dSkYj561DX7s1CpBuOBt
```

ssh -C -p 2220 bandit21@bandit.labs.overthewire.org

passwd:EeoULMCra2q0dSkYj561DX7s1CpBuOBt

## Level 22

```
cd /etc/cron.d/
cat cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q
```

ssh -C -p 2220 bandit22@bandit.labs.overthewire.org

passwd:tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q

## Level 23

```
cd /etc/cron.d/
cat cronjob_bandit23
cat /usr/bin/cronjob_bandit23.sh

echo I am user bandit23 | md5sum | cut -d ' ' -f 1
8ca319486bfbbc3663ea0fbe81326349

cat /tmp/8ca319486bfbbc3663ea0fbe81326349
0Zf11ioIjMVN551jX3CmStKLYqjk54Ga
```

ssh -C -p 2220 bandit23@bandit.labs.overthewire.org

passwd:0Zf11ioIjMVN551jX3CmStKLYqjk54Ga

## Level 24

```
cd /etc/cron.d/
cat cronjob_bandit24

cd /var/spool/bandit24/foo
echo -e '#!/bin/bash\nmkdir -p /tmp/wwww/; cp /etc/bandit_pass/bandit24 /tmp/wwww/pass.txt; chmod 777 /tmp/wwww/pass.txt' > ok.sh && chmod +x ok.sh

cat /tmp/wwww/pass.txt
gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8
```

ssh -C -p 2220 bandit24@bandit.labs.overthewire.org

passwd:gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8

## Level 25

```
for pass in {0000..9999}; do echo "gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 $pass"; done | nc localhost 30002 | grep -v 'Wrong'

The password of user bandit25 is iCi86ttT4KSNe1armKiwbQNmB3YJP3q4
```

ssh -C -p 2220 bandit25@bandit.labs.overthewire.org

passwd:iCi86ttT4KSNe1armKiwbQNmB3YJP3q4

## Level 26

```
cat /etc/passwd | grep 26
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext

cat /usr/bin/showtext
#!/bin/sh

export TERM=linux

exec more ~/text.txt
exit 0

/usr/bin/showtext 中执行了more命令，可以将窗口调小，然后使more可以显示分屏滚动，这时就可以使用:!sh得到shell，阻止 exit 0

cat bandit26.sshkey

-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEApis2AuoooEqeYWamtwX2k5z9uU1Afl2F8VyXQqbv/LTrIwdW
pTfaeRHXzr0Y0a5Oe3GB/+W2+PReif+bPZlzTY1XFwpk+DiHk1kmL0moEW8HJuT9
/5XbnpjSzn0eEAfFax2OcopjrzVqdBJQerkj0puv3UXY07AskgkyD5XepwGAlJOG
xZsMq1oZqQ0W29aBtfykuGie2bxroRjuAPrYM4o3MMmtlNE5fC4G9Ihq0eq73MDi
1ze6d2jIGce873qxn308BA2qhRPJNEbnPev5gI+5tU+UxebW8KLbk0EhoXB953Ix
3lgOIrT9Y6skRjsMSFmC6WN/O7ovu8QzGqxdywIDAQABAoIBAAaXoETtVT9GtpHW
qLaKHgYtLEO1tOFOhInWyolyZgL4inuRRva3CIvVEWK6TcnDyIlNL4MfcerehwGi
il4fQFvLR7E6UFcopvhJiSJHIcvPQ9FfNFR3dYcNOQ/IFvE73bEqMwSISPwiel6w
e1DjF3C7jHaS1s9PJfWFN982aublL/yLbJP+ou3ifdljS7QzjWZA8NRiMwmBGPIh
Yq8weR3jIVQl3ndEYxO7Cr/wXXebZwlP6CPZb67rBy0jg+366mxQbDZIwZYEaUME
zY5izFclr/kKj4s7NTRkC76Yx+rTNP5+BX+JT+rgz5aoQq8ghMw43NYwxjXym/MX
c8X8g0ECgYEA1crBUAR1gSkM+5mGjjoFLJKrFP+IhUHFh25qGI4Dcxxh1f3M53le
wF1rkp5SJnHRFm9IW3gM1JoF0PQxI5aXHRGHphwPeKnsQ/xQBRWCeYpqTme9amJV
tD3aDHkpIhYxkNxqol5gDCAt6tdFSxqPaNfdfsfaAOXiKGrQESUjIBcCgYEAxvmI
2ROJsBXaiM4Iyg9hUpjZIn8TW2UlH76pojFG6/KBd1NcnW3fu0ZUU790wAu7QbbU
i7pieeqCqSYcZsmkhnOvbdx54A6NNCR2btc+si6pDOe1jdsGdXISDRHFb9QxjZCj
6xzWMNvb5n1yUb9w9nfN1PZzATfUsOV+Fy8CbG0CgYEAifkTLwfhqZyLk2huTSWm
pzB0ltWfDpj22MNqVzR3h3d+sHLeJVjPzIe9396rF8KGdNsWsGlWpnJMZKDjgZsz
JQBmMc6UMYRARVP1dIKANN4eY0FSHfEebHcqXLho0mXOUTXe37DWfZza5V9Oify3
JquBd8uUptW1Ue41H4t/ErsCgYEArc5FYtF1QXIlfcDz3oUGz16itUZpgzlb71nd
1cbTm8EupCwWR5I1j+IEQU+JTUQyI1nwWcnKwZI+5kBbKNJUu/mLsRyY/UXYxEZh
ibrNklm94373kV1US/0DlZUDcQba7jz9Yp/C3dT/RlwoIw5mP3UxQCizFspNKOSe
euPeaxUCgYEAntklXwBbokgdDup/u/3ms5Lb/bm22zDOCg2HrlWQCqKEkWkAO6R5
/Wwyqhp/wTl8VXjxWo+W+DmewGdPHGQQ5fFdqgpuQpGUq24YZS8m66v5ANBwd76t
IZdtF5HXs2S5CADTwniUS5mX1HO9l5gUkk+h0cH5JnPtsMCnAUM+BRY=
-----END RSA PRIVATE KEY-----
```

ssh -i id_rsa -C -p 2220 bandit26@bandit.labs.overthewire.org 将窗口调小，触发 more 的分屏模式

按下 v 键进入编辑模式, :set shell=/bin/bash :shell 就得到了 shell 的执行环境。

```
cat /etc/bandit_pass/bandit26
s0773xxkk0MXfdqOfPRVr9L3jJBUOgCZ
```

## Level 27

```
bandit26@bandit:~$ ./bandit27-do cat /etc/bandit_pass/bandit27
upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB
```

ssh -C -p 2220 bandit27@bandit.labs.overthewire.org

passwd:upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB

## Level 28

```
mkdir -p /tmp/27 && cd /tmp/27
git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo

passwd:upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB

cd repo
The password to the next level is: Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN
```

ssh -C -p 2220 bandit28@bandit.labs.overthewire.org

passwd:Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN

## Level 29

```
mkdir -p /tmp/28 && cd /tmp/28
git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repo

passwd:Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN

cd repo
git log
git show 0fe4ed2645761a54f3bad315b4899c385319d8f7

password: 4pT1t5DENaYuqnqvadYs1oE4QLCdjmJ7
```

ssh -C -p 2220 bandit29@bandit.labs.overthewire.org

passwd:4pT1t5DENaYuqnqvadYs1oE4QLCdjmJ7

## Level 30

```
mkdir -p /tmp/29 && cd /tmp/29
git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repo

passwd:4pT1t5DENaYuqnqvadYs1oE4QLCdjmJ7

cd repo
git log
git show

git branch
git branch -r
git checkout dev
cat README.md
password: qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL
```

ssh -C -p 2220 bandit30@bandit.labs.overthewire.org

passwd:qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL

## Level 31

```
mkdir -p /tmp/30 && cd /tmp/30
git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repo

passwd:qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL

cd repo
git branch -a # no branch

git tag
git show secret

fb5S2xb7bRyFmAvQYQGEqsbhVyJqhnDy
```

ssh -C -p 2220 bandit31@bandit.labs.overthewire.org

passwd:fb5S2xb7bRyFmAvQYQGEqsbhVyJqhnDy

## Level 32

```
mkdir -p /tmp/31 && cd /tmp/31
git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo

passwd:fb5S2xb7bRyFmAvQYQGEqsbhVyJqhnDy

cd repo
echo 'May I come in?' > key.txt
rm .gitignore
git add .
git commit -m 'ok'
git push origin master

remote: Well done! Here is the password for the next level:
remote: 3O9RfhqyAlVBEZpVb6LYStshZoqoSx5K
```

ssh -C -p 2220 bandit32@bandit.labs.overthewire.org

passwd:3O9RfhqyAlVBEZpVb6LYStshZoqoSx5K

## Level 33

bandit32:x:11032:11032:bandit level 32:/home/bandit32:/home/bandit32/uppershell

```
输入的内容被转换成了大写字母，然后用bash -c 执行的命令，$0 代表当前的执行脚本或shell名，所以可以绕过upper

$0
id
cat /etc/bandit_pass/bandit33
tQdtbs5D5i2vJwkO8mEyYEyTL8izoeJ0
```

ssh -C -p 2220 bandit33@bandit.labs.overthewire.org

passwd:tQdtbs5D5i2vJwkO8mEyYEyTL8izoeJ0

## Level 34

```
bandit33@bandit:~$ cat README.txt
Congratulations on solving the last level of this game!

At this moment, there are no more levels to play in this game. However, we are constantly working
on new levels and will most likely expand this game with more levels soon.
Keep an eye out for an announcement on our usual communication channels!
In the meantime, you could play some of our other wargames.

If you have an idea for an awesome new level, please let us know!
```

over ~~~
