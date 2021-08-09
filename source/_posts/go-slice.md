---
title: go-切片
date: 2019-6-13 22:10:00
tags:
- go
- 切片
categories:
- golang
preview: 100
---

[数组和切片区别](https://segmentfault.com/a/1190000012326168)
数组容量固定，不可伸缩，相同维数和长度的数组可以比较，但是切片不可比较


## 切片

### 声明


```go
var a []int
var b []string{}

fmt.println(a == nil)
fmt.println(b == nil)
```
输出结果为`true, false`

原因是:
1. 声明但是未使用的切片默认值是nil
2. 已经给b分配了内存空间，所以不为空

### 使用make()构造切片
make函数的原型是`make([]T, size, cap)`
```go
c := make([]int, 2, 10)
d := make([]int, 2)
fmt.Println(c == nil, d == nil)
```
输出结果肯定是false, false 因为使用make函数一定发生了内存分配

### 扩容
Go 语言的内建函数 append() 可以为切片动态添加元素。当空间不能容纳足够多的元素时，切片就会进行“扩容”。“扩容”操作往往发生在 append() 函数调用时。
容量的扩展规律按容量的 2 倍数扩充，例如 1、2、4、8、16……，代码如下：

```go
var numbers []int

for i := 0; i < 10; i++ {
    numbers = append(numbers, i)
    fmt.Printf("len: %d  cap: %d pointer: %p\n", len(numbers), cap(numbers), numbers)
}
```
输出结果
``` 
len: 1  cap: 1 pointer: 0xc0420080e8
len: 2  cap: 2 pointer: 0xc042008150
len: 3  cap: 4 pointer: 0xc04200e320
len: 4  cap: 4 pointer: 0xc04200e320
len: 5  cap: 8 pointer: 0xc04200c200
len: 6  cap: 8 pointer: 0xc04200c200
len: 7  cap: 8 pointer: 0xc04200c200
len: 8  cap: 8 pointer: 0xc04200c200
len: 9  cap: 16 pointer: 0xc042074000
len: 10  cap: 16 pointer: 0xc042074000
```

切片的cap大小取决于原数组，比如：
```go
var a [...]int{1, 2, 3, 4, 5}
b := a[2:4]
```
b的cap不是4-2, 而是从原数组2的位置到末尾，所以cap应该是5-2=3


### append()函数

``` go
var car []string

// 添加一个元素
append(car, "Ice")

// 添加多个元素
append(car, "BMW", "Monk")

// 连接两个切片
x := []int {1,2,3}
y := []int {4,5,6}

fmt.Println(car)
fmt.Println(appebd(x, y...))
```

### 复制切片内容

```go
	c := make([]int, 10)
	for i:=0; i<10; i++ {
		c[i] = i*i
	}

	// 直接赋值只是引用而不是复制
	refData := c
	println(refData[2])
	c[2] = 0;
	println(refData[2])

	println("----------")

	// 使用copy函数
	copyData := make([]int, 9)
	copy(copyData, c)
	c[3] = 0
	println(copyData[3])
	// println(copyData[9]) 报错
```
1. 直接赋值只是引用而不是复制
2. copy函数并不会自动使copyData扩容

### 删除切片元素
Go 语言并没有对删除切片元素提供专用的语法或者接口，需要使用切片本身的特性来删除元素。

```go
	a := []int{1, 2, 3, 4, 5}
	// 删除3
	index := 2
	a = append(a[:index], a[index+1:]...)
	fmt.Printf("%X", a)
```
