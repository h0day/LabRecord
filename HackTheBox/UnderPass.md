# UnderPass

2025.03.06 https://app.hackthebox.com/machines/UnderPass

## Ip

10.10.11.48

## Scan

Tcp 开放 22、80，Udp 开放 snmp 161

```
snmpwalk -v 2c -c public 10.10.11.48
snmpwalk -v 2c -c public 10.10.11.48
```

`UnDerPass.htb is the only daloradius server in the basin` 将 underpass.htb 添加到 hosts
