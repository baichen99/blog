---
title: go 面向对象
date: 2019-6-13 22:10:00
tags:
- go
- object oriented
- 面向对象
categories:
- golang
---

## 封装数据和方法

### 结构体

```golang
type Employee struct {
  ID    int
  Name  string
}
```

初始化:
```golang
e := Employee(1, "bob")
e1 := Employee(ID: 2, Name: "bob")
e2 := new(Employee) // 这里返回的是指针
e2.ID = 3  // 指针可以直接用.访问属性
```

#### 结构体可以内嵌

- 可以内嵌，而且只有字段的类型是必须的
- 内嵌的结构体可以直接访问其成员变量
  结构体实例访问任意一级的嵌入结构体成员时都只用给出字段名，而无须像传统结构体字段一样，通过一层层的结构体字段访问到最终的字段。例如，ins.a.b.c的访问可以简化为ins.c。
- 内嵌结构体的字段名是它的类型名
- [初始化](http://c.biancheng.net/view/74.html)

```golang
type Base struct {
	ID string
}

type Employee struct {
	Base
  	Name  string
	int
}



func main() {
  // 初始化
	e := &Employee{Base: Base{ID: "12313"}, Name: "bob", int :4}
	e1 := new(Employee)
	e1.ID = "123"
	e1.Name = "Amy"
	fmt.Println(e)
	fmt.Println(e.int)
	fmt.Println(e1)
}
```

### 实现方法

```golang
func (e Employee) String() string {
  return fmt.Sprintf("ID: %d - Name: %s", e.ID)
}
```

```golang
func (e *Employee) String() string {
  return fmt.Sprintf("ID: %d - Name: %s", e.ID)
}
```

第一种定义的方法在调用的时候会复制实例对象, 为了避免内存拷贝选择第二种

### interface

- 因为接口类型不会和特定的实现细节绑定在一起，通过这种抽象的方式我们可以让我们的函数更加灵活和更具有适应能力。

- 如果一个任意类型 T 的方法集为一个接口类型的方法集的超集，则我们说类型 T 实现了此接口类型。T 可以是一个非接口类型，也可以是一个接口类型。实现关系在Go语言中是隐式的。两个类型之间的实现关系不需要在代码中显式地表示出来

- 接口类型命名一般以er结尾

- 定义函数的时候参数定义成 interface，调用函数的时候就可以做到非常的灵活以。

- 函数的参数为interface时，调用函数要传入strcut的指针类型的原因是，*S实现了方法，而不是S。

- 尽可能依赖较小的接口，这样才能在更多的地方复用。

以下代码中，file struct实现了WriteData方法，所以file类型也是DataWriter类型，所以才可以 `writer = f`

```golang
package main

import (
    "fmt"
)

// 定义一个数据写入器
type DataWriter interface {
    WriteData(data interface{}) error
}

// 定义文件结构，用于实现DataWriter
type file struct {
}

// 实现DataWriter接口的WriteData方法
func (d *file) WriteData(data interface{}) error {

    // 模拟写入数据
    fmt.Println("WriteData:", data)
    return nil
}

func main() {

    // 实例化file
    f := new(file)

    // 声明一个DataWriter的接口
    var writer DataWriter

    // 将接口赋值f，也就是*file类型
    writer = f

    // 使用DataWriter接口进行数据写入
    writer.WriteData("data")
}
```

- 一个类型可以同时实现多个接口，而接口间彼此独立，不知道对方的实现。
- 多个嵌套结构体实现共同实现一个接口，使用者并不关心某个接口的方法是通过一个类型完全实现的，还是通过多个结构嵌入到一个结构体中拼凑起来共同实现的。

#### 类型断言

[类型断言](https://juejin.im/post/6844904153056034823)

#### 空接口

因为所有类型都实现了空接口的方法，所以所有类型都是空接口类型。
可以给空接口随意赋值，但不可以随意取出（要进行类型断言） [http://c.biancheng.net/view/84.html](http://c.biancheng.net/view/84.html)




