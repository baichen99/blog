---
title: flask01-路由
date: 2018-09-22 12:33:48
tags: 
  - flask 
  - route 
  - 路由
categories: 
  - flask学习笔记
preview: 100
---

### route装饰器
route()装饰器的第一个参数是**URL规则**，用字符串表示，必须以斜杠开始，这里的URL是相对URL。 
以`www.blacston.com`为例，'/' 对应的是根地址 `www.blacton.com`, '/hello'对应的是 `www.blacton.com/hello`
<!-- more -->
### 为视图函数绑定多个URL
一个视图函数可以绑定多个URL
```python
@app.route('/hi')
@app.route('/hello')
def say_hello():
    return 'hello'
```
这样访问两个URL时都会出发say_hello()函数

### 动态URL
```python
@app.route('/greet/<name>')
def greet(name):
    return 'hello %s' % name

```
Flask会解析请求并把请求的URL与视图函数的URL规则进行匹配，比如这里的URL规则是/greet/<name>
那么/greet/baichen，/greet/bar 的请求都会出发这个视图函数

### 设置默认值
当URL规则包含变量时，如果用户访问的url没有变量，flask在匹配失败后会返回一个404错误响应，解决方法是在route()函数里使用defaults参数设置变量的默认值，这个参数接收字典作为输入
```python
@app.route('/hello', defaults={'name': '游客'})
@app.route('/hello/<name>')
def hello(name):
    return 'hello %s' % name

```
它其实相当于
```python
@app.route('/hello')
@app.route('/hello/<name>')
def hello(name='游客'):
    return 'hello %s' % name
    
```

### URL处理
```python
@app.route('/hello/<int:year>')
def hello(year):
    return 'year is : ' % year

```
其中int把year转换成了整型，flask还提供了其他的转换器，有以下几种

- string
- int
- float
- path  （包含斜线的字符串）
- any   匹配一系列给定值的一个元素
- uuid

any的用法如下
```python
@app.route('/colors/any(blue, red, white):color')
def three_colors(color):
    return 'i like %s' % color

```