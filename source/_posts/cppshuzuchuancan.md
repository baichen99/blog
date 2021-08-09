---
title: C++ 数组传参
date: 2018-11-08 22:18:12
tags: 
  - 数组
categories: 
  - C++ 
preview: 100
---

**在C/C++中，在进行数组传参时，c虽然传递的是收地址，但是在函数内部就成了普通指针，不是数组首地址了，所以在函数里得到数组长度是不可以的，所以函数参数中有数组时一般要传入数组大小**



### 错误示范

```C++
int GetLen(int *arr)
return sizeof(arr)/sizeof(*a);
```

### 正确例子

1. `void func(int *a, int length){}`

2. `void func(int a[length]`

3. `void func(int a[], int length){}`

### 参考文章
[C/C++数组传参](https://www.cnblogs.com/spring-hailong/p/6110685.html)
[C/C++数组参数传递后，还能计算长度吗？](https://blog.csdn.net/u013025203/article/details/54379104)

