---
title: XSS跨站脚本攻击
date: 2021-01-10 11:56:48
tags:
- web
- xss
categories:
- web
preview: 100
---

# XSS

## 定义

跨站脚本攻击(Cross Site Sript)是指攻击者利用网站对**用户输入过滤不足**，输入内容可以**显示在页面上**对其他人造成影响的HTML代码，利用用户身份进行某种动作，从而完成盗取用户资料、进行病毒侵害的一种攻击方式。为了和层叠样式表(Cascading Style Sheets)的缩写区分，跨站脚本攻击通常简写为XSS。

## 反射型

原理：简单的将用户输入的数据直接或未经过完善的**安全过滤**就在浏览器中进行输出，导致输出的数据中存在可被浏览器执行的代码数据，由于此种类型的跨站代码存在于URL中，所以黑客通常需要诱骗或加密变形等方式将存在恶意代码的链接发给用户，**只有用户点击以后才能使得攻击成功实施**。

假如有个程序是这样：

```python
# 伪代码
@route('/')
def index(request):
	name = request.args.get('name')
  return "hello" + name
```

那么当访问'/?name='Bob'时，网页会显示hello bob。但这个程序对用户输入内容没有过滤和转义，也就是说我们可以将一些js代码写在参数中，这样服务器返回的响应中就会包含着写代码。比如当把name的值改为js代码，也就是访问`/?name=<script>alert(1)</script>`的时候，页面执行了`alert(1)`，然后会出现一个弹窗。所以黑客会把这段url藏到超链接中，然后诱导用户去点击链接，就能完成XSS攻击。

## 存储型

原理：存储型XSS脚本攻击是指由于Web应用程序对用户输入数据的处理不严格，导致Web应用程序将黑客输入的恶意跨站攻击数据信息保存在服务端的数据库或其他文件中，当王爷进行数据查询展示时，会从数据库中获取数据内容，并将数据内容在网页中进行输出展示，进而导致XSS代码执行。

常见场景：留言板、博客、新闻发布系统中，恶意代码的数据信息直接写入文章、评论中。

```python
# 伪代码
@route('/')
def index(request):
  # 返回所有消息
	message_list = Message.objects.all()
  return 200, {message_list: message_list}

@route('/', method=['POST'])
def index(request):
	message = request.form.get('msg')
  # 数据库中新增一条记录
  m = Message(text=message)
  m.save()
  # 201 表示创建成功
  return 201
```



## 反射型和存储型对比

反射型和存储型是根据表现形式来区分的，其实他们的本质都是一样的：网页可以直接显示用户输入的内容。区别在于：存储型xss指的是用户输入内容可以保存在服务器中，可以影响到其他用户的。

## 预防方法

- 对输入、输出信息进行过滤和转义
  - 表单验证，对符合格式的输入允许通过检查。比如邮件地址必须是xx@xx.xx格式，age必须是int类型。
  - 对特殊字符进行过滤：比如HTML中的`" ' < > % &`
- 使用[HttpOnly](https://blog.csdn.net/qq_38553333/article/details/80055521), 将重要的cookie标记为httponly，这样的话当浏览器向Web服务器发起请求的时就会带上`cookie`字段，**但是在`js`脚本中却不能访问这个cookie**，这样就避免了XSS攻击利用`JavaScript`的`document.cookie`获取`cookie`。



## 参考内容：

[视频演示-XSS跨站脚本漏洞原理](https://www.bilibili.com/video/BV1WK411V7Sz)

[对于跨站脚本攻击（XSS攻击）的理解和总结](https://zhuanlan.zhihu.com/p/37295186)