# sunset: dawn2

2025.03.03 https://www.vulnhub.com/entry/sunset-dawn2,424/

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
80/tcp   open  http
1435/tcp open  ibm-cics
1985/tcp open  hsrp
```

访问 web 提示下载http://192.168.5.40/dawn.zip , 解压后是 dawn.exe，readme 说这是服务器端程序。应该就是上面开放的另外 2 个端口中的一个，需要对这个 exe 进行逆向。

nc 连接到 1985 端口，发现输入信息时会进行回显，这里可能存在缓冲区溢出，进行测试，输入 300 个 A 发现程序直接退出了。

在 windows 上对这个 dawn.exe 进行动态调试

待续...
