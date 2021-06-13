---
layout:     post
title:      初探 Golang ReverseProxy 源码
subtitle:   初探 Golang ReverseProxy 源码
date:       2021-06-13
author:     锐玩道
header-img: img/bg_img/post-bg-cook.jpg
catalog:      true
theme:      smartblue
tags:
    - golang
---

> 如果❤️我的文章有帮助，欢迎点赞、关注。这是对我继续技术创作最大的鼓励。[更多系列文章在我博客](https://coderdao.github.io/)

# 初探 Golang ReverseProxy 源码

> Golang 版本 1.16

## 使用 ReverseProxy 实现反向代理的例子
Talk is cheap, show me the code
![图片描述](http://img1.sycdn.imooc.com/60c64c6a0001499219681716.png)

测试 代理服务器
```bash
$ curl 'http://127.0.0.1:2002/sda?sda=111'

# 2003 设置了 打印请求地址代码： upath := fmt.Sprintf("http://%s%s\n", r.Addr, req.URL.Path)
http://127.0.0.1:2003/base/sda
RemoteAddr=127.0.0.1:51738,X-Forwarded-For=127.0.0.1,X-Real-Ip=
headers =map[Accept:[*/*] Accept-Encoding:[gzip] User-Agent:[curl/7.69.1] X-Forwarded-For:[127.0.0.1]]
```

查看 `httputil.NewSingleHostReverseProxy()` 函数, 便可以找到 `ReverseProxy`源码。 位于该路径下文件`go/src/net/http/httputil/reverseproxy.go`

# 核心源码
## ReverseProxy 结构体
![图片描述](http://img1.sycdn.imooc.com/60c64c35000144f519681436.png)


## ReverseProxy 处理请求的方法 ServeHTTP()

ReverseProxy 实现了`ServeHTTP()` 方法, 最终会在`请求到达代理服务器`, 由监听方法`ListenAndServe()`调起。调用链路为：
```text
http.ListenAndServe(addr string, handler Handler)  
-> Server.ListenAndServeTLS(certFile, keyFile) 
-> Server.ServeTLS(ln, certFile, keyFile)
-> Server.Serve(l net.Listener)
-> go c.serve(connCtx)
-> serverHandler{c.server}.ServeHTTP(w, w.req)
```

所以 ReverseProxy 结构体也实现了 `ServeHTTP` 方法 , 方法实现功能有：
1. 拷贝上游请求的 context 到下游请求  
2. 使用 指定 director（对请求进行修改的函数）修改请求（例如协议、参数、url 等）  
3. 根据请求 Header["Connection"] 判断是否需要升级协议（Upgrade）  
4. 删除上游请求中的 hop-by-hop Header, 维持上游持久(相对)连接, 不需要透传到下游  
5. 设置 X-Forward-For Header，追加当前节点 IP  
6. 使用连接池，向下游发起请求  
7. 处理 httpcode 101 协议升级：（WebSocket、h2c 等）  
8. 删除请求中的 hop-by-hop Header, 不要返回给上游
9. 根据结构体 ReverseProxy.ModifyResponse（函数）判断是否修改响应体内容
10. 拷贝下游响应头部到上游响应请求  
11. 返回 下游请求 HTTP 状态码
12. 拷贝 下游响应内容 到 上游响应请求  
13. 刷新内容到 response

![图片描述](http://img1.sycdn.imooc.com/60c64bb300017b0d19687129.png)

# 其他源码

## NewSingleHostReverseProxy 函数
![图片描述](http://img1.sycdn.imooc.com/60c64c0b00014b5319681576.png)

## NewSingleHostReverseProxy 中 URL 拼接方法
![图片描述](http://img1.sycdn.imooc.com/60c64be900011df519680835.png)
