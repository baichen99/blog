---
title: go func
date: 2019-6-13 22:10:00
tags:
- go
- function
categories:
- golang
---

## 特点
1. 多个返回值
2. 参数传递都是值传递
3. 函数可以作为变量值
4. 函数可以作为参数和返回值

## 可变参数
参数数量可变
```go
func sum(ops ...int)int {
  res = 0
  for _, op := range ops {
    res += op
  }
  return res
}
```

## defer

1. defer会在函数执行完成后调用
2. panic不会影响defer的执行