---
title: 指针,引用和常量
date: 2018-09-13 07:44:15
categories: 
  - C++
tags: 
  - 指针 
  - 引用 
  - 常量 
  - C++
cover: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1594188669721&di=ae97828bf15801fad0959278f5c29c7a&imgtype=0&src=http%3A%2F%2Fimg.mp.itc.cn%2Fq_mini%2Cc_zoom%2Cw_640%2Fupload%2F20170704%2Fa9e90a651c9d4ba5b8472382bbc753a0_th.jpg
preview: 100
---


### 常量指针和指针常量

**常量指针**是指向常量的指针，如：`const int* p` 其中p是一个指针，指向了一个常量，也就是`*p`不能修改，而p的值可以修改

**指针常量**是个常量，如：`int *const p` p是个指针，p（指针的指向）不能修改，但是`*p`可以修改

**指向常量的指针常量**，如：`const int const* p` p是个指针常量，而且指向的值也是个常量
<!-- more -->

#### 应用举例

字符串处理函数的函数的声明。它们的参数一般声明为常量指针。例如，字符串比较函数的声明是这样的：
` int strcmp(const char *str1, const char *str2); `
这样做的目的是，函数的参数声明用了常量指针的形式，保证了在函数内部，那个常量不被更改，
可以接受**非常量**的字符串，这是因为变量可以当作常量，而常量不可以当作变量，比如下面这段代码就是错误的

```
const int a = 5;
int* p = &a;
```

### 引用和常量引用

```
int a =10;

cout<<"b:"<<endl;
const int& b = a;
cout<<b<<' ';
a = 20;
cout<<b<<endl;


cout<<"c: "<<endl;
const int& c = a*2;
cout<<c<<' ';
a=30;
cout<<c<<endl;
```

输出结果

```
b:
10 20
c: 
40 40
```
可以看出来，当常量引用的初始值是变量时，可以通过变量修改常量，当初始值是const时是不能修改的。

```
double val = 3.14;
const int &r4 = val;
std::cout << "r4 = " << r4 << std::endl;
val = 5.2;
std::cout << "r4 = " << r4 << "     val = " << val << std::endl;
```

输出结果

```
r4 =3
r4 = 3       val = 5.2
```

这种情况似乎与r2一样，但是仔细观察就会发现val是`double`型，而r4是`int`型，
所以我们可以知道：当常量引用的类型和它的初始值的类型不同时，无法通过变量修改引用值。

#### 使用引用常量传递参数
如果不想让函数修改原来的变量，可以使用传值和传入常量引用的方法

```
upper(char* a);
upper(char* const &a);
```

这两种方法那个更好呢？答案是使用常量引用，因为使用传值会在函数内部创建一个副本，这样会降低效率。



### 参考资料
[常量指针与指针常量的区别-flyge](https://www.cnblogs.com/FlyGee/p/7424852.html)<br>
[尽量使用“引用常量”传递函数参数](https://blog.csdn.net/liupeng1985/article/details/23534485)<br>
[对const的引用(常量引用)](https://www.cnblogs.com/yang666/p/6546966.html)

