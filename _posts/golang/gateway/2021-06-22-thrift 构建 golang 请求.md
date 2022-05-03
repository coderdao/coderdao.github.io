---
layout:     post
title:      thrift 构建 golang 请求
subtitle:   thrift 构建 golang 请求
date:       2021-06-17
author:     锐玩道
header-img: img/bg_img/post-bg-cook.jpg
catalog:      true
theme:      smartblue
tags:
    - golang
---

> 如果❤️我的文章有帮助，欢迎点赞、关注。这是对我继续技术创作最大的鼓励。[更多系列文章在我博客]   coderdao.github.io

# thrift 构建 golang 请求

## 简介
其实今天本来是想写 更硬核的 thrift 构建 tcp 代理，结果 thrift 0.11.0 版本暂无办法安装。导致一大堆代码报错。只能退而求其次，说说 `thrift 构建 golang 请求` 

## 环境
window 10

## 安装 thrift
[thrift 官网]上 window 有直接的 exe 文件；而 linux/mac 则需要编译安装

![QQ截图20210621235107.png](http://img3.sycdn.imooc.com/60d0b5c60001bf3606700378.jpg)

而如果需版本有要求的话，thrift 也提供历史版本，但你必须确认 `安装 thrift 版本号` 与 程序使用 `thrift SDK 版本号` 保持一致。否则之后 thrift `IDL生成命令` 生成代码 会因为报错而无法运行

安装成功后，运行命令检查是否安装成功：
```bash
$ /d/Dev/env/thrift014/thrift014.exe -version
Thrift version 0.14.2
```
## 下载 thrift 源码
```bash
git clone git@github.com:apache/thrift.git

# 如果 github 网速慢也可以使用国内镜像
git clone git@gitee.com:apache/thrift.git
```

打开文件夹我们可以找到各种语言版本代码，这里主要说的是 golang，所以我们选择`go文件夹`
![图片描述](//img1.sycdn.imooc.com/60d0b941000135ce07270495.png)

不过可惜的是，我目前还没找到 go module 离线安装的方法。所以依旧使用在线安装方式 `go get git.apache.org/thrift.git` 安装 thrift

## 环境搭建
 
 在 目录 `thrift/tutorial` 我们可以找到 模型例子，具体操作如下
 
 1、在 `$GOROOT/src` 目录下，新建文件夹 `thrift_server_client` 并且 启用 go mod
 ```bash
$  mkdir $GOROOT/src/thrift_server_client
$  cd $GOROOT/src/thrift_server_client
$  go mod init thrift_server_client
 ```
 
 2.、 安装 `thrift SDK` 
  ```bash
$  go get git.apache.org/thrift.git
 ```
 
 3、 复制 `thrift 源码` 目录 `thrift/tutorial` 下 文件 `shared.thrift`、`tutorial.thrift` 到 `$GOROOT/src/thrift_server_client` 文件下
![图片描述](//img1.sycdn.imooc.com/60d0be060001f96406760487.png)

4、运行IDL生成命令，生成对应语言 server/client 结构体  'thrift --gen [语言] [代码配置文件]` 
```bash
$  /d/Dev/env/thrift014/thrift014.exe --gen go shared.thrift
$  /d/Dev/env/thrift014/thrift014.exe --gen go tutorial.thrift
```
便可以得到下面的代码
![图片描述](//img1.sycdn.imooc.com/60d0becf00019bc003750407.png)

5、如果你这时有阅读生成的代码，就会发现 `tutorial.go`，`tutorial-consts.go` 存在的 `shared` 引用是找不到的（idle 无法阅读源码）
![图片描述](//img1.sycdn.imooc.com/60d0bf7d00018a3a12500460.png)

其实，`shared`它指向的就是我们刚才 `/d/Dev/env/thrift014/thrift014.exe --gen go shared.thrift` 生成的代码。所以把 `shared` 引用替换为相对路径 `thrift_server_client/gen-go/shared` 即可

## 构建 服务端、客户端
我们再把 `thrift/tutorial/go/src` 拷贝至 `$GOROOT/src/thrift_server_client` 文件下

只要看 `main.go` 文件
```go
	// 是否服务端开启
	server := flag.Bool("server", false, "Run server")
	// 指定请求协议
	protocol := flag.String("P", "binary", "Specify the protocol (binary, compact, json, simplejson)")
	// 绑定监听地址
	addr := flag.String("addr", "localhost:9090", "Address to listen to")
	// 是否开启https
	secure := flag.Bool("secure", false, "Use tls secure transport")
``` 

- 首先 server := flag.Bool("server",`true`, "Run server") 打开服务端，运行
```bash
$ go run main.go server.go client.go handler.go
*thrift.TServerSocket
Starting the simple server... on  localhost:9090
```

- 然后 启动客户端 server := flag.Bool("server",`false`, "Run server") ，运行
```bash
$ go run main.go server.go client.go handler.go
ping()
1+1=2
Invalid operation: InvalidOperation({WhatOp:4 Why:Cannot divide by 0})
15-10=5
Check log: 5
```

这是服务端也会接受到客户端同学，打印内容：
```bash
Starting the simple server... on  localhost:9090
ping()
add(1,1)
calculate(1, {DIVIDE,1,0})
calculate(1, {SUBTRACT,15,10})
getStruct(1)
```

至此，如何使用 `thrift` 示例请求到此结束，之后如长链接、wss 也能够基于该示例改造而来