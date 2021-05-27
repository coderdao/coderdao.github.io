---
layout:     post
title:      iris 基于 ab 或 wrk 的性能测试
subtitle:   iris 基于 ab 或 wrk 的性能测试
date:       2021-04-26
author:     锐玩道
header-img: img/bg_img/post-bg-cook.jpg
catalog:      true
theme:      smartblue
tags:
    - golang
---


> 如果❤️我的文章有帮助，欢迎点赞、关注。这是对我继续技术创作最大的鼓励。[更多往期文章在我的个人博客](https://coderdao.github.io/)

# 基于 ab 或 wrk的样例测试

- apache 自带 ad
- wrk 需要github 安装：https://github.com/wg/wrk

ab默认短链接，wrk默认长连接

## ab
`ab` 全称 `apache bench`。

使用ab命令时，它会创建`多个并发访问线程`，模拟多个用户 `同时` 对一链接地址`发起请求`。由于是对`链接地址（URL）`为目标进行测试，所以它可以用来测试`apache`、`nginx`、`tomcat`、`IIS`等等Web服务器的负载压力。

ab命令对 `发起请求端` 负载很低，不会占用太多的CPU、内存资源；但对`压测服务器`会造成巨大负载，其原理类似CC网络攻击。

个人做测试的时候尤其需要注意：压测的目标是为了找出`压测服务器`的**负载上限、性能瓶颈（以资源消耗70%为基准）**；

而不是盲目发起过多请求，消耗目标表服务器资源甚至`宕机`。这属于故意破坏他人计算机系统，严重情况下目标服务器公司会根据请求日志进行追责。

### 安装
命令 `ab` 为 `apache` 服务器自带工具。如果不想安装 `apache` 又要使用命令 `ab`，可以支架安装 `apache` 工具包 `httpd-tools`：
> yum -y install httpd-tools

## ab 命令说明
```
ab: wrong number of arguments
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     总共测试多少请求
    -c concurrency  并发请求数量
    -t timelimit    测试进行时间（秒）。默认：无时间限制下，进行 5w 请求
    -s timeout      每个请求等待响应时间，默认：30秒
    -b windowsize   TCP 发送/接收 缓存区尺寸（bytes）
    -B address      Address to bind to when making outgoing connections
    -p postfile     File containing data to POST. Remember also to set -T
    -u putfile      File containing data to PUT. Remember also to set -T
    -T content-type Content-type header to use for POST/PUT data, eg.
                    'application/x-www-form-urlencoded'
                    Default is 'text/plain'
    -v verbosity    How much troubleshooting info to print
    -w              Print out results in HTML tables
    -i              Use HEAD instead of GET
    -x attributes   String to insert as table attributes
    -y attributes   String to insert as tr attributes
    -z attributes   String to insert as td or th attributes
    -C attribute    Add cookie, eg. 'Apache=1234'. (repeatable)
    -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip'
                    Inserted after all normal header lines. (repeatable)
    -A attribute    Add Basic WWW Authentication, the attributes
                    are a colon separated username and password.
    -P attribute    Add Basic Proxy Authentication, the attributes
                    are a colon separated username and password.
    -X proxy:port   Proxyserver and port number to use
    -V              Print version number and exit
    -k              Use HTTP KeepAlive feature
    -d              Do not show percentiles served table.
    -S              Do not show confidence estimators and warnings.
    -q              Do not show progress when doing more than 150 requests
    -l              Accept variable document length (use this for dynamic pages)
    -g filename     Output collected data to gnuplot format file.
    -e filename     Output CSV file with percentages served
    -r              Don't exit on socket receive errors.
    -m method       Method name
    -h              Display usage information (this message)
    -I              Disable TLS Server Name Indication (SNI) extension
    -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)
    -f protocol     Specify SSL/TLS protocol
                    (SSL3, TLS1, TLS1.1, TLS1.2 or ALL)
```

### 测试例子
```
ab -n10000 -c10 http://localhost:8080/index

Server Software:        
Server Hostname:        localhost
Server Port:            8080

Document Path:          /index
Document Length:        4866 bytes

Concurrency Level:      10
Time taken for tests:   13.309 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      49620000 bytes
HTML transferred:       48660000 bytes

Requests per second:    751.37 [#/sec]  - qps -每秒响应请求数

Time per request:       13.309 [ms] (mean)  - 每个请求耗时
Time per request:       1.331 [ms] (mean, across all concurrent requests)
Transfer rate:          3640.93 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       8
Processing:     3   13   8.3     13     280
Waiting:        2   13   8.3     13     280
Total:          3   13   8.3     13     280

Percentage of the requests served within a certain time (ms)
  50%     13
  66%     13
  75%     13
  80%     14
  90%     14
  95%     15
  98%     15
  99%     16
 100%    280 (longest request)
```

## wrk
> wrk is a modern HTTP benchmarking tool capable of generating significant load when run on a single multi-core CPU. It combines a multithreaded design with scalable event notification systems such as epoll and kqueue.
-- https://github.com/wg/wrk

和 上述 ab 相似，不同地方在于 `wrk` 在单机多核 CPU 环境下，能够使用`系统自带`高性能 I/O 机制：epoll(linux)，kqueue(mac)，通过`多线程`和`事件模式`更少的线程就制造出更大的并发。

### wrk 安装
```
mac：brew install wrk

linux:

git clone https://github.com/wg/wrk.git  
cd wrk && make
ln -s [wrk安装路径]/wrk /usr/local/bin  # 在任何路径直接使用wrk
```

```
Usage: wrk <options> <url>                            
  Options:                                            
    -c, --connections <N>  并发量   
    -d, --duration    <T>  压测时间           
    -t, --threads     <N>  开多少线程压测   
                                                      
    -s, --script      <S>  指定Lua脚本路径       
    -H, --header      <H>  为每一个HTTP请求添加HTTP头      
        --latency          在压测结束后，打印延迟统计信息   
        --timeout     <T>  超时时间     
    -v, --version          打印正在使用的wrk的详细版本信息      
                                                      
  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)
```

### 测试例子
```
wrk -c10 -t10 -d5 http://localhost:8080/index

$ wrk -c10 -t10 -d5 http://localhost:8080/index
Running 5s test @ http://localhost:8080/index         压测时间5s 
  10 threads and 10 connections                       10个测试线程，10个连接
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    12.79ms    1.19ms  21.37ms   77.94%    延迟
    Req/Sec    78.18      5.42    90.00     72.60%    每秒请求数
  3922 requests in 5.03s, 18.76MB read                5.03s内处理3922请求，耗费流量73.86MB
Requests/sec:    779.06       - qps -每秒响应请求数
Transfer/sec:      3.73MB     - 每秒流量
```
