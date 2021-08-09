---
title: python 深浅拷贝 值传递 引用传递
date: 2021-01-14 19:09:25
tags:
- python
- 深拷贝
- 浅拷贝
categories:
- python

---


## 可变对象 不可变对象

参考内容:

 [python可变对象与不可变对象](https://zhuanlan.zhihu.com/p/34395671)

[Python的函数参数传递：传值？引用？](http://winterttr.me/2015/10/24/python-passing-arguments-as-value-or-reference/)

可变对象：list dict set
不可变对象：tuple string int float bool

可变对象修改后地址不会改变，不可变对象修改后地址会改变。因为修改不可变对象时，原来的对象被丢弃，变量指向新的对象。修改可变对象时，比如修改列表中的第一个元素，是有一个新的对象被指定给列表对象的第一个元素，但是列表本身没有变化，只是内容发生了变化。

```python
nfoo = 1
nfoo = 2

lstFoo = [1]
lstFoo[0] = 2
```

代码第2行中，内存中原始的1对象因为不能改变，于是被“抛弃”，另nfoo指向一个新的int对象，其值为2

代码第5行中，更改list中第一个元素的值，因为list是可改变的，所以，第一个元素变更为2。其实应该说，lstFoo指向一个`包含一个对象的数组`。赋值所发生的事情，是有一个新int对象被指定给lstFoo所指向的数组对象的第一个元素，但是对于lstFoo本身来说，所指向的数组对象并没有变化，只是数组对象的内容发生变化了。这个看似void*的变量所指向的对象仍旧是刚刚的那个有一个int对象的list。

![](https://i.loli.net/2021/01/14/RkraAjZYH2gJtKq.jpg)



所以函数传参的时候不要传可变对象，否则可能会影响传入对象的值。




深浅拷贝的例子都可以在[这个网站](http://pythontutor.com/)上可视化编程，非常直观。

## 赋值

赋值是将一个对象的地址赋值给一个变量，让变量指向该地址。
修改不可变对象（int, str、tuple）需要开辟新的空间
修改可变对象（list等）不需要开辟新的空间

```python
a = [1, 2, 3, [1, 2]]
b = a
b[0] = 2
```
a和b两个变量指向的内存空间完全相同，此时对a修改会影响b的值。
![](https://i.loli.net/2021/01/14/qQBzXxC2iUbEsYo.png)


## 浅拷贝
浅拷贝是在另一块地址中创建一个新的变量或容器，但是容器内的元素的地址均是源对象的元素的地址的拷贝。也就是说新的容器中指向了旧的元素（ 新瓶装旧酒 ）。
```python
a = [1, 2, 3, [1, 2]]
b = a.copy()
```
![](https://i.loli.net/2021/01/14/EqosgdjKfwL6Tva.png)

## 深拷贝
深拷贝完全拷贝了一个副本，容器内部元素地址都不一样

```python
import copy
a = [1, 2, 3, [1, 2]]
b = copy.deepcopy(a)
```

[![sdcv8K.png](https://s3.ax1x.com/2021/01/14/sdcv8K.png)](https://imgchr.com/i/sdcv8K)

