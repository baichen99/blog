---
title: go string
date: 2019-6-13 22:10:00
tags:
- go
- string
categories:
- golang
---

## 定义赋值初始化

string是不可变的只读的[]byte类型
unicode是一种字符集
UTF-8是unicode的一种实现，是变长编码

```go
var s string
var str string = "中国"
var a = "中国"
```

len(str)获取的是字节数，不一定是字符数

通过下标访问得到的是字节，需要进行转换或者格式化打印
```go
package main

import (
    "fmt"
)

func main() {
    str := "世界"
    //方法一：格式化打印
    for _, ch1 := range str {
        fmt.Printf("%q",ch1) //单引号围绕的字符字面值，由go语法安全的转义
    }
    
    //方法二：转化输出格式
    for _, ch2 := range str {
        fmt.Println(string(ch2))
    }
}
```

## strings包
[strings](https://golang.org/pkg/strings/)

## strconv包
[strconv](https://golang.org/pkg/strconv/#example_AppendBool)

## string常见用法
[http://c.biancheng.net/view/17.html](http://c.biancheng.net/view/17.html)

## rune 和 byte 区别
[https://www.cnblogs.com/wanghui-garcia/p/10568354.html](https://www.cnblogs.com/wanghui-garcia/p/10568354.html)