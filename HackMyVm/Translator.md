# Translator

2025.02.05 https://hackmyvm.eu/machines/machine.php?vm=Translator

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

访问 80 web 首页显示的是一个翻译页面，输入一个 5 时，会显示 2 个 5，这里可能会出现漏洞点。

先 gobuster 扫描看看有没有其他隐藏目录，没有发现：

```
gobuster -t 32 dir -u http://192.168.5.40/ -k -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -e -x txt,php,html,jpg
```

仔细研究下这个翻译功能吧，
