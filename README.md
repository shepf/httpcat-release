English | [简体中文](translations/README-cn.md)

## 🚀HttpCat Overview
HttpCat is an HTTP file transfer service implemented in Go, designed to provide a simple, efficient, and stable solution for file uploading and downloading.

Project goals: To create a reliable, efficient, and user-friendly HTTP file transfer Swiss Army Knife that greatly enhances your control and experience with file transfers. Whether it's for temporary sharing or bulk file transfers, HttpCat will be your excellent assistant.

Please note that this translation is a direct translation and may require further refinement by a professional translator for the best results.


## 💥Key Features
* Simple and easy to use
* No external dependencies, easy to port

## 🎉Installation
Extract to the httpcat directory:
```bash
tar -zxvf httpcat_*.tar.gz -C httpcat
```

Modify configuration file:
```bash
cd httpcat
vi ./conf/svr.yml
```

linux:
```bash
./httpcat -C conf/svr.yml
```

windows:
```bash
httpcat.exe --upload /home/web/website/download/ --download /home/web/website/download/ -C F:\open_code\httpcat\server\conf\svr.yml
```

```bash
# ./httpcat -h
Usage of ./httpcat:
  -C, --config string     ConfigPath (default "./conf/svr.yml")
      --download string   Specify the path for downloading files, ending with a forward slash (/) (default "./website/download/")
  -P, --port int          host port.
      --static string     Specify the path for static resources (web), ending with a forward slash (/) (default "./website/static/")
      --upload string     Specify the path for uploading files, ending with a forward slash (/) (default "./website/upload/")
```


### Run in the background using tmux
You can use tmux to run in the background:
```bash
Create a new tmux session using a socket file named tmux_httpcat
$ tmux -S tmux_httpcat

#  Once inside tmux, you can execute running commands, such as:
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


### Linux can use systemd to run in the background
```bash
cp  httpcat.service /usr/lib/systemd/system/httpcat.service
systemctl daemon-reload
systemctl start httpcat
```

> Note: You may need to modify the startup parameters according to your needs. 
> Ensure that the following three directories are consistent (so that the upload directory is also the download directory, 
> and it is also the web frontend directory where files can be downloaded without authentication).

```bash
vi httpcat.service
```
```
ExecStart=/usr/local/bin/httpcat  --static=/home/web/website/upload/  --upload=/home/web/website/upload/ --download=/home/web/website/upload/  -C /etc/httpdcat/svr.yml
```


## ❤ Tips and Tricks
### File Operation Related APIs
#### Uploading Files Using Curl Tool
```bash
curl -vF "f1=@/root/hello.mojo" http://localhost:8888/api/v1/file/upload
```
The curl command is used to send a multipart/form-data format POST request to the specified URL. Here is an explanation of each part:
- `curl`:  curl is a tool used for transferring data to/from a server, supporting multiple protocols.
- `-v`: Detailed operational information is displayed during command execution, which is known as verbose mode.
- `-F "f1=@/root/hello.mojo"`: Specifies the form data to be sent. The -F option indicates that a form is being sent, and f1=@/root/hello.mojo indicates that the file field to be uploaded is named f1, with the file path /root/hello.mojo. The value of this field is the relative or absolute path to the local file.
- `http://localhost:8888/api/v1/file/upload`: 要发送请求到的 URL，这条命令会将文件上传到这个 URL。

> Note: f1 is defined in the server-side code. Modifying it to something else, such as file, will result in an error and the upload will fail.


#### File Upload Authentication
If the enable_upload_token option is enabled in the configuration file, file uploads require authentication. You need to add a token in the request header, where the token value should match the enable_upload_token value specified in the configuration file.
An independent upload token credential is generated based on the app_key and app_secret. When uploading a file, include the token in the request. The server will then verify the validity of the token.



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


```bash
curl -v -F "f1=@/root/hello.mojo" -H "UploadToken: httpcat:bbE8NVvimYNbV-CaJ9EFMKg3YaM=:eyJkZWFkbGluZSI6M15=" http://localhost:8888/api/v1/file/upload
```

####  Upload file Enterprise WeChat webhook notification.
Configure the persistent_notify_url in the svr.yml file. After a successful upload, an Enterprise WeChat notification will be sent.

The notification message is as follows:

File upload archived, upload information:
- IP Address: 192.168.31.3
- Upload Time: 2023-11-29 23:07:04
- File Name: syslog.md
- File Size: 4.88 KB
- File MD5: 8346ecb8e6342d98a9738c5409xxx

#### Support SQLite to retain upload history.
If the enable_sqlite option is enabled in the configuration, uploaded files will be recorded in an SQLite database. You can use the `sqlite` command-line tool to query the upload history records.

Use the `sqlite` command-line tool to create a database and query data.

```bash
sudo apt install sqlite3
sqlite3 --version
```

Run the following command to connect to an SQLite database and specify the filename for the database to be created (e.g., sqlite.db):
```bash
sqlite3 sqlite.db
```

At the sqlite3 prompt, enter the .tables command to list all the tables in the database:
```bash
.tables
```

```bash
SELECT * FROM notifications;
```


#### 下载文件
##### api 接口
View the list of files in a specific directory within the download root.
`http://127.0.0.1:8888/api/v1/file/listFiles?dir=
`
Download a specific file.
`http://127.0.0.1:8888/api/v1/file/download?filename=xxx.jpg
`
Get information about a specific file, including its MD5 hash.
`http://{{ip}}:{{port}}/api/v1/file/fileInfo?name=FlF9mrjXgAAZHon.jpg
`


### P2P Related APIs
P2P functionality needs to be enabled in the configuration file, which is disabled by default.

#### Sending messages to the P2P network via HTTP API
http://{{ip}}:{{port}}/api/v1/p2p/send_message
POST
{
"topic": "httpcat",
"message": "ceshi cccccccccccc"
}

