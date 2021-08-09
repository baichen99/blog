---
title: go map
date: 2019-6-13 22:10:00
tags:
- go
- map
categories:
- golang
---

## 声明赋值初始化
```golang
var a = map[string]int{"one": 1, "two": 2}
var b = map[string]int{}
b["age"] = 1
c = make(map[string]int, 10) // 10为容量
```

## 遍历

```go
var a = map[string]int{"one": 1, "two": 2}
for k, v := range a {
  fmt.Println(k, v)
}
```

## 其他要注意的地方

### 访问不存在的key会返回0值

不能通过判断是否为nil来判断是否存在
如何判断是0还是不存在？

```go
var a = map[string]int{"one": 1, "two": 2}
if v, ok := a[3]; ok {
  fmt.Printf("%d's value exists", v)
} else {
  fmt.Printf("%d's value does not exists", v)
}
```

### map的value可以是一个方法

```go
func TestMap(t *testing.T) {
  m := map[string]func(op int) int{}
  m1 := func(op int) int { return op }
  m2 := func(op int) int { return op*op }
  m["one"] = m1
  m["two"] = m2
  fmt.Println(m["one"](4), m["two"](4))
}
```

### 用map实现set()

1. 元素唯一
`map[type] bool`
2. 添加
`m[1]=true`
3. 判断存在
`if m[1]`
4. 删除
`delete(m, 1)`
5. 个数
`len(m)`
