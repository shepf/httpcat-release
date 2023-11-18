[English](README.md) | 简体中文
## 🚀HttpCat 概述 

HttpCat 是一个基于go实现的 HTTP 的文件传输服务，旨在提供简单、高效、稳定的文件上传和下载功能。

项目目标：一个可靠、高效、易用的HTTP文件传输瑞士军刀,它将大大提高你的文件传输控制力和体验。
无论是临时分享还是批量传输文件,HttpCat都将是你的优秀助手。

## 💥功能特点
* 简单易用
* 无需外部依赖，易于移植

## 🎉安装 
下载：


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
./httpcat -c conf/svr.yml
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
httpcat --static=/home/web/website/upload/  --c server/conf/svr.yml

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
注意：你可能需要根据你的需要修改启动参数,如下 3个目录一致:
```bash
vi httpcat.service
```
```
ExecStart=/usr/local/bin/httpcat  --static=/home/web/website/upload/  --upload=/home/web/website/upload/ --download=/home/web/website/upload/  -C /etc/httpdcat/svr.yml
```


## ❤使用技巧
### 使用curl工具上传文件
```bash
curl -vF "f1=@/root/hello.mojo" http://localhost:8888/api/v1/file/upload
```
使用了 `curl` 命令来向指定的 URL 发送一个 `multipart/form-data` 格式的 POST 请求。下面是对每部分的解释：
- `curl`: 一个用来与服务器端进行数据传输的工具，支持多种协议。
- `-v`: 在命令执行时输出详细的操作信息，即 verbose 模式。
- `-F "f1=@/root/hello.mojo"`: 指定了要发送的表单数据。`-F` 选项表示要发送一个表单，`f1=@/root/hello.mojo` 表示要上传的文件字段名为 `f1`，文件路径为 `/root/hello.mojo`。这个字段的值是指向本地文件的相对或绝对路径。
- `http://localhost:8888/api/v1/file/upload`: 要发送请求到的 URL，这条命令会将文件上传到这个 URL。

> 注意： f1 为服务端代码定义的，修改为其他，如file，会报错上传失败。


### 下载文件
#### api 接口
查看下载根目录下，某个目录的文件列表
`http://127.0.0.1:8888/api/v1/file/listFiles?dir=
`
下载某个具体的文件
`http://127.0.0.1:8888/api/v1/file/download?filename=xxx.jpg
`
获取某个文件的信息，包括md5
`http://{{ip}}:{{port}}/api/v1/file/fileInfo?name=FlF9mrjXgAAZHon.jpg
`

## 💪TODO
1. 接口增加签名认证机制
   将请求方法、URL、查询字符串、访问密钥、时间戳、请求体的哈希值等信息按照一定规则拼接起来，并使用给定的密钥进行签名计算。
   生成的签名用于在请求头部或其他方式中进行身份验证或安全控制。
2. 支持p2p环境下http使用
3. https支持


欢迎提issue~ Good luck 🍀
