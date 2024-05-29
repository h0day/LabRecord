# Glasgow Smile: 1.1

2024-5-x https://www.vulnhub.com/entry/glasgow-smile-11,491/

difficulty: Intermediate

## IP

192.168.10.176

## Scan

Open Port -> 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 6734481f250ed7b3eabb361122608fa1 (RSA)
|   256 4c8c4565a484e8b1507777a93a960631 (ECDSA)
|_  256 09e994236097f720cceed6c19bda188e (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
```

http://192.168.10.176/ 首页是图片，其他什么都没有，图片上也没有隐写信息。

gobuster 扫描一下：

```
gobuster -t 64 dir -u http://192.168.10.176/ -k -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,html -e

http://192.168.10.176/joomla               (Status: 301) [Size: 317] [--> http://192.168.10.176/joomla/]
http://192.168.10.176/how_to.txt           (Status: 200) [Size: 456]
```

how_to.txt 先看看显示什么，可能存在一个叫 rob 的账号。

另外还有一个 joomla 的 cms，访问 http://192.168.10.176/joomla/robots.txt 得到了一些目录信息：

```
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

http://192.168.10.176/joomla/administrator/ 登陆页面，先用前面得到的 rob 用户，尝试下爆破。
