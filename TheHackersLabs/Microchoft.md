# Microchoft

2025.03.16 https://thehackerslabs.com/microchoft/

## Ip

192.168.5.39

## Scan

```
PORT      STATE    SERVICE
135/tcp   open     msrpc
139/tcp   open     netbios-ssn
445/tcp   open     microsoft-ds
12658/tcp filtered unknown
46166/tcp filtered unknown
47195/tcp filtered unknown
49152/tcp open     unknown
49153/tcp open     unknown
49154/tcp open     unknown
49155/tcp open     unknown
49156/tcp open     unknown
49157/tcp open     unknown
```

发现目标操作系统 Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1) 可能存在 eternalblue ，尝试使用 msfconsole 进行利用，直接拿到了 system 权限，拿到 flag:

```
c:\Users>type c:\users\Lola\Desktop\user.txt
13e624146d31ea232c850267c2745caa

c:\Users>type c:\users\Admin\Desktop\admin.txt.txt
ff4ad2daf333183677e02bf8f67d4dca
```
