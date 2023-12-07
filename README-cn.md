[English](README.md) | 简体中文
## 🚀HttpCat 概述

HttpCat 是一个基于go实现的 HTTP 的文件传输服务，旨在提供简单、高效、稳定的文件上传和下载功能。

项目目标：一个可靠、高效、易用的HTTP文件传输瑞士军刀,它将大大提高你的文件传输控制力和体验。
无论是临时分享还是批量传输文件,HttpCat都将是你的优秀助手。

## 💥功能特点
* 简单易用
* 无需外部依赖，易于移植

## 🎉安装
解压到httpcat目录:
```bash
tar -zxvf httpcat_*.tar.gz -C httpcat
```

修改配置文件:
```bash
cd httpcat
vi ./conf/svr.yml
```

linux下运行:
```bash
./httpcat -C conf/svr.yml
```

windows下运行:
```bash
httpcat.exe --upload /home/web/website/download/ --download /home/web/website/download/ -C F:\open_code\httpcat\server\conf\svr.yml
```

```bash
# ./httpcat -h
Usage of ./httpcat:
  -C, --config string     ConfigPath (default "./conf/svr.yml")
      --download string   指定下载文件的路径,右斜线结尾 (default "./website/download/")
  -P, --port int          host port.
      --static string     指定静态资源路径(web) (default "./website/static/")
      --upload string     指定上传文件的路径,右斜线结尾 (default "./website/upload/")
```

### 使用tmux运行在后台
可以利用tmux方式后台运行:
```bash
Create a new tmux session using a socket file named tmux_httpcat
$ tmux -S tmux_httpcat

# 进入tmux后，可以执行运行命令,如：
httpcat --static=/home/web/website/upload/  -C server/conf/svr.yml

Move process to background by detaching
Ctrl+b d OR ⌘+b d (Mac)

To re-attach
$ tmux -S tmux_httpcat attach

Alternatively, you can use the following single command to both create (if not exists already) and attach to a session:
$ tmux new-session -A -D -s tmux_httpcat

To delete farming session
$ tmux kill-session -t tmux_httpcat
```

### linux可以使用systemd运行在后台
```bash
cp  httpcat.service /usr/lib/systemd/system/httpcat.service
systemctl daemon-reload
systemctl start httpcat
```

> 注意：根据你的业务需要修改启动参数
> 比如：一个最简单的应用场景：3个目录一致(则上传目录就是下载目录，并且也是web前端目录，可以无认证直接下载):
```bash
vi httpcat.service
```
```
ExecStart=/usr/local/bin/httpcat  --static=/home/web/website/upload/  --upload=/home/web/website/upload/ --download=/home/web/website/upload/  -C /etc/httpdcat/svr.yml
```


## ❤使用技巧
### 文件操作相关接口
#### 使用curl工具上传文件
```bash
curl -vF "f1=@/root/hello.mojo" http://localhost:8888/api/v1/file/upload
```
使用了 `curl` 命令来向指定的 URL 发送一个 `multipart/form-data` 格式的 POST 请求。下面是对每部分的解释：
- `curl`: 一个用来与服务器端进行数据传输的工具，支持多种协议。
- `-v`: 在命令执行时输出详细的操作信息，即 verbose 模式。
- `-F "f1=@/root/hello.mojo"`: 指定了要发送的表单数据。`-F` 选项表示要发送一个表单，`f1=@/root/hello.mojo` 表示要上传的文件字段名为 `f1`，文件路径为 `/root/hello.mojo`。这个字段的值是指向本地文件的相对或绝对路径。
- `http://localhost:8888/api/v1/file/upload`: 要发送请求到的 URL，这条命令会将文件上传到这个 URL。

> 注意： f1 为服务端代码定义的，修改为其他，如file，会报错上传失败。


#### 上传文件认证
如果配置文件开启了 `enable_upload_token`，那么上传文件需要认证，需要在请求头中添加token，token的值为配置文件中的`enable_upload_token`值。
根据app_key、app_secret生成独立的上传token凭证。上传文件时候，附带token，服务端会校验token是否合法。


http://{{ip}}:{{port}}/api/v1/user/createUploadToken
POST
{
"accessKey": "httpcat",
"secretKey": "httpcat_app_secret"
}
例如返回：
{
"code": 0,
"msg": "success",
"data": "httpcat:dZE8NVvimYNbV-YpJ9EFMKg3YaM=:eyJkZWFkbGluZSI6MH0="
}


You can use the -H option to add custom HTTP headers in the cURL command.
```bash
curl -v -F "f1=@/root/hello.mojo" -H "UploadToken: httpcat:bbE8NVvimYNbV-CaJ9EFMKg3YaM=:eyJkZWFkbGluZSI6M15=" http://localhost:8888/api/v1/file/upload
```

#### 上传文件企业微信webhook通知
配置svr.yml文件中的`persistent_notify_url`，上传成功后，会发送企业微信通知。

通知信息如下：
```
有文件上传归档,上传信息：
- IP地址：192.168.31.3
- 上传时间：2023-11-29 23:07:04
- 文件名：syslog.md
- 文件大小：4.88 KB
- 文件MD5：8346ecb8e6342d98a9738c5409xxx
```

#### 支持sqlite保留上传历史记录
如果配置开启了 enable_sqlite，那么上传文件会记录到sqlite数据库中，可以通过sqlite3命令行工具查询上传历史记录。


使用sqlite3命令行工具创建数据库，查询数据
```bash
sudo apt install sqlite3
sqlite3 --version
```

运行以下命令启动 sqlite3 工具，并指定要创建的数据库文件名（例如 mydatabase.db）：
```bash
sqlite3 sqlite.db
```

在 sqlite3 提示符下，输入 `.tables` 命令来列出数据库中的所有表：
```bash
.tables
```

```bash
SELECT * FROM notifications;
```

#### 下载文件
##### api 接口
查看下载根目录下，某个目录的文件列表
`http://127.0.0.1:8888/api/v1/file/listFiles?dir=
`
下载某个具体的文件
`http://127.0.0.1:8888/api/v1/file/download?filename=xxx.jpg
`
获取某个文件的信息，包括md5
`http://{{ip}}:{{port}}/api/v1/file/fileInfo?name=FlF9mrjXgAAZHon.jpg
`

### p2p相关接口
需要配置文件开启p2p功能，默认关闭

#### 通过http接口向p2p网络发送消息
http://{{ip}}:{{port}}/api/v1/p2p/send_message
POST
{
"topic": "httpcat",
"message": "ceshi cccccccccccc"
}

## 💪TODO
1. HTTPS support
2. Internationalization


Feel free to raise an issue. Good luck! 🍀

