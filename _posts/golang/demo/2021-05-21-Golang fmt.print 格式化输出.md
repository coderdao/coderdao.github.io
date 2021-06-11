---
layout:     post
title:      Golang fmt.print 格式化输出
subtitle:   Golang fmt.print 格式化输出
date:       2021-05-11
author:     锐玩道
header-img: img/bg_img/post-bg-cook.jpg
catalog:      true
theme:      smartblue
tags:
    - golang
---


> 如果❤️我的文章有帮助，欢迎点赞、关注。这是对我继续技术创作最大的鼓励。[更多往期文章在我的个人博客](https://coderdao.github.io/)

Golang fmt 标准包 **功能类似于C语言printf** 实现了格式化I/O获取、输出。
源自C语言的实现但Golang的使用起来更简单，功能主要分为 `输出内容` 和 `获取输入` 两大部分。

获取输入 已经在 [Golang fmt.Scan 获取输入]() 详细讲述过，这里就不再纠结。接下来具体列举 fmt 向外输出

Golang fmt 标准包提供了几种输出方式如下：

## Print
Print() 相关函数会`标准化将内容输出到系统`，区别在于：
- Print() 直接输出内容，语法如下：
    > func Print(a ...interface{}) (n int, err error)
- Printf() 支持输出格式化字符串，语法如下：
    > func Printf(format string, a ...interface{}) (n int, err error)
- Println() 在输出内容的结束后换行，语法如下：
    > func Println(a ...interface{}) (n int, err error)

具体例子：
```go
package main

import (
	"fmt"
)

func main() {
    fmt.Print("在命令行打印该信息。")
    name := "Go语言"
    fmt.Printf("我是：%s\n", name)
    fmt.Println("在命令行打印单独一行显示")
}

/**
执行代码输出：
    在命令行打印该信息。我是：Go语言
    在命令行打印单独一行显示
*/
```

## Fprint

Fprint() 相关函数会把内容输出到 `io.Writer接口类型` 的变量中，`常用于打印日志到文件中`。

区别在于：
- Fprint() 直接输出内容输出到变量中，语法如下：
    > func Fprint(w io.Writer, a ...interface{}) (n int, err error)
- Fprintf() 根据规格格式化内容输出到变量中，语法如下：
    > func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
- Fprintln() 输出内容的末尾添加换行符输出到变量中，语法如下：
    > func Fprintln(w io.Writer, a ...interface{}) (n int, err error)

具体例子：
```go
package main

import (
	"fmt"
)

func main() {
    fmt.Fprintln(os.Stdout, "标准输出内容到文件")
    fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        fmt.Println("打开文件出错，err:", err)
        return
    }

    name := "Go语言"
    // 向文件句柄中写入内容, 注意，只有满足io.Writer接口的类型才支持写入。
    fmt.Fprintf(fileObj, "往文件写入信息：%s", name)
}

/**
执行代码输出内容到文件 xx.txt：
    往文件写入信息：Go语言
*/
```


## Sprint
Sprint() 相关函数会把传入的数据生成并返回一个字符串，常用语格式化字符串，如：大段文字中替换对应变量。

区别在于：
- Sprint() 直接返回内容，语法如下：
    > func Sprint(a ...interface{}) string
- Sprintf() 根据格式格式化内容并返回，语法如下：
    > func Sprintf(format string, a ...interface{}) string
- Sprintln() 根据格式格式化内容结尾添加换行符，语法如下：
    > func Sprintln(a ...interface{}) string

具体例子：
```go
package main

import (
	"fmt"
)

func main() {
    s1 := fmt.Sprint("Go语言")
    name := "Go语言"
    age := 18
    s2 := fmt.Sprintf("name:%s,age:%d", name, age)
    s3 := fmt.Sprintln("Go语言")
    fmt.Println(s1, s2, s3)
}

/** 
执行代码输出内容
    Go语言 name:Go语言,age:18 Go语言
*/
```
