# CryptoLabyrinth

2025.02.27 https://thehackerslabs.com/cryptolabyrinth/

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

扫描出 http://192.168.5.40/hidden/ 有一些哈希文件和 salt 文件，挨个看一下：

```
http://192.168.5.40/hidden/pista_aes.txt

This is where your quest begins. The AES key is well hidden.
```

```
http://192.168.5.40/hidden/numeros_suerte.txt

Important: Lucky numbers are 7, 14, 21. The key to decrypt Alice is supercomplexkey!
```

```
http://192.168.5.40/hidden/informe_segur_bob.txt

Critical Security Report: Don't Look Here. (提示无用)
```

```
http://192.168.5.40/hidden/importante_pista_alice.txt

Critical clue: Alice's password is encrypted.
```

```
http://192.168.5.40/hidden/datos_sensibles_alice.txt

Alice's Sensitive Data: This file has no clues. (提示无用)
```

```
http://192.168.5.40/hidden/clue_bob.txt

Bob's private key is fragmented. Gather all the parts to decrypt his password!
```

```
http://192.168.5.40/hidden/clue_aes.txt

Find the hash to crack Alice's password...
```

```
http://192.168.5.40/hidden/bob_salt_hash.txt

6f9f059ee11b1c409d403e0c72bd4929546987afca3530b3c8975f513b3e15a2
```

```
http://192.168.5.40/hidden/bob_salt.txt

d5882bc2e9ca61f9
```

```
http://192.168.5.40/hidden/bob_password1.hash
e10adc3949ba59abbe56e057f20f883e

http://192.168.5.40/hidden/bob_password2.hash
c378985d629e99a4e86213db0cd5e70d

http://192.168.5.40/hidden/bob_password3.hash
0d107d09f5bbe40cade3de5c71e9e9b7

http://192.168.5.40/hidden/bob_password4.hash
d8578edf8458ce06fbc5bb76a58c5ca4

http://192.168.5.40/hidden/bob_password5.hash
93ad6b16ba90629960e76a2c718ff4a5
```

还有一个 enc 文件 http://192.168.5.40/hidden/alice_aes.enc

从上面的信息中可以看到 2 个账户 bob 和 alice
