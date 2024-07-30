# Vulnerable Docker: 1

2024-8-x https://www.vulnhub.com/entry/vulnerable-docker-1,208/

difficulty: begginer-intermediate

## IP

192.168.5.37

## Scan

pen Port -> 22,8000

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6p1 Ubuntu 2ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 45130881706d46c350ed3cabaed6e185 (DSA)
|   2048 4ce72b0152161d5c6b099d3d4bbb7990 (RSA)
|   256 cc2f62714cea6ca6d8a74feb822a22ba (ECDSA)
|_  256 73bfb4d6ad51e3992629b742e3ffc381 (ED25519)
8000/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: NotSoEasy Docker &#8211; Just another WordPress site
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-generator: WordPress 4.8.1
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
```
