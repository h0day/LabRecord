# Paella

2025.03.03 https://thehackerslabs.com/paella/

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
10000/tcp open  snet-sensor-mgmt
```

MiniServ 1.920 (Webmin httpd) 存在 unauthenticated RCE，使用这个 exp https://github.com/ruthvikvegunta/CVE-2019-15107/blob/master/webmin_rce.py
