---
layout:     post
title:      Golang init() 函数
subtitle:   Golang init() 函数
date:       2021-06-03
author:     锐玩道
header-img: img/bg_img/post-bg-cook.jpg
catalog:    true
theme:      smartblue
tags:
    - golang
---

> 如果❤️我的文章有帮助，欢迎点赞、关注。这是对我继续技术创作最大的鼓励。[更多往期文章在我的个人博客](https://coderdao.github.io/)


# Golang init() 函数

## 先举个例子

```go
package main

import "fmt"

func init()  {
	fmt.Println("init() 1:", a)
}

func init()  {
	fmt.Println("init() 2:", a)
}

var a = 10
const b = 100

func main() {
	fmt.Println("main() :", a)
}

// 执行结果
// init() 1: 10
// init() 2: 10
// main() : 10
```

## init() 是什么
在 `Go` 语言设计过程中保留了默认的两个函数，分别是 `main()` 和 `init()` 函数。

两者的区别在于：
- `main()` 函数只能使用于 `main` 包中，而且每个 `main` 包只能有 `一个main()` 函数
- 但对于 `init()` 函数, 则能够使用在所有的包中。而且一个程序（甚至一个文件）中可以写任意多个 `init()` 函数。
> 注意：一个程序（甚至一个文件）中可以写任意多个 `init()` 函数，但对于维护代码`可读性`、`排查问题`并没有任何好处

## init() 特点
- `init()` 用于 `程序运行前` 的进行包初始化（自定义变量、实例化通信连接）工作
- `每个包`、`每个程序文件` 可以同时拥有多个init()，但`不建议`
- 同一个包、文件中多个 `init() 执行顺序`, Golang 中并未明确
- 不同包的 `init()`执行顺序，按照 `导入包的依赖关系` 决定
- `init()` 不能被其他函数调用，而自动 `在main函数执行前` 被调用

—— 参考来源于 [effective_go](https://golang.google.cn/doc/effective_go#init)

## init() 什么时候执行
`init()` 函数 是 Golang `程序初始化` 包含的一部分。

在 Golang 中程序的 `初始化先于 main()` 执行：具体由 `runtime` 初始化每个被导入的包。
- 初始化顺序是按照`解析的依赖关系`的顺序执行，**没有依赖的包最先初始化**。
- 首先初始化的是 每个包作用域内的`常量`、`变量`（其中：常量先于变量），之后执行包内 `init()`。
- 相同一个包、文件可以`同时拥有`多个 init()。
- init() 和 main() 一样，`没有任何参数和返回值`，不能够被其他函数调用。
- 同一个包、文件 `多个 init()` 执行顺序并未明确。

执行顺序总结： import –> const –> var –> init() –> main()
