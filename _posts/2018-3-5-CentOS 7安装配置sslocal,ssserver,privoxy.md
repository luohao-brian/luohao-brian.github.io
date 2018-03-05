---
layout: post
title: CentOS 7 安装配置sslocal, ssserver, privoxy
tags: [Linux]
---

### 服务器
安装shadowsocks:
```
#server
pip install shadowsocks
```
shadowsocks服务器端配置文件server.json:
```
{
    "server_port":26100,
    "local_port":1080,
    "password":"密码",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
启动服务:
```
ssserver -c ss/server.json -d start
```

### 客户端

安装**shadowsocks**和**privoxy**:

#shadowsocks
```
pip install shadowsocks
#privoxy
yum install -y privoxy
```
**shadowsocks**客户端配置文件client.json

```
{
    "server":"服务器IP",
    "server_port":26100,
    "local_port":1080,
    "password":"密码",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
启动shadowsocks客户端服务，该socks服务默认监听1080端口:
```
sslocal -c ss/client.json -d start
```
**privoxy**配置（不要漏掉最后的.符号），该服务默认监听8118端口，以下配置会将来自8118端口的http请求转发到本地1080端口：
```
# /etc/privoxy/config
forward-socks5t   /               127.0.0.1:1080 .
启动privoxy服务
privoxy /etc/privoxy/config
```

设置代理

```
proxy () {
  export http_proxy="http://127.0.0.1:8118"
  export https_proxy="http://127.0.0.1:8118"
  echo "HTTP Proxy on"
}
# where noproxy
noproxy () {
  unset http_proxy
  unset https_proxy
  echo "HTTP Proxy off"
}
```