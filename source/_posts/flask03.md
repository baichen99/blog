---
title: flask03-cookies, session
date: 2018-09-23 10:58:00
tags:
- flask
- cookies
- session
categories:
- flask学习笔记
preview: 100
---

使用session和cookies制作一个简单的登录系统

### cookie

#### 设置cookie
在flask中，在响应中添加cookie最方便的方法是使用Response类提供的`set_cookie()`方法，要使用这个方法需要先试用`make_response()`方法生成一个response对象

set_cookie()方法支持多个参数来设置Cookie的选项

---

key cookie的键
value  cookie的值
max_age   cookie保持的时间，单位为秒；默认在用户关闭浏览器过期
expires   具体过期时间
path
domain
secure
httponly

---

#### 获取cookie
cookie可以通过request的cookies属性来获取
```python
@app.route('/')
@app.route('/hello')
def hello():
    name = request.args.get('name')
    if name is None:
        name = request.cookies.get('name', '游客')
       return 'hello %s' %s name

```
### session：更安全的cookie

#### 设置程序密钥
session用来将cookie数据加密储存，所以需要设置一个密钥，是一个随机复杂的字符串
1. 通过Flask.secret_key或配置变量SECRET_KEY设置
`app.secret_key= 'secret string'`
2. 更安全的办法是将其写入环境变量中(在命令行使用`export`/`set`命令),或者保存在.env文件中
然后用os.getenv()方法获取
```python
import os
...
# 第二个参数是没有获取到SECRET_KEY取的默认值
app.secret_key = os.getenv('SECRET_KEY', 'secret string')
```
#### 模拟用户登录
```python
from flask import redirect, session, url_for
...
@app.route('/login')
def login():
    session['logged_in'] = True
    return redirect(url_for('hello'))

```
在hello视图中:
```python
@app.route('/hello')
def hello():
    # 从query string里获取name参数
    name = request.args.get('name')
    if name is None:
        name = request.cookies.get('name', '游客')
    # escape 防止XSS攻击
    response = 'hello %s' % escape(name)
    if session.get('logged_in'):
        response += '[已登录]'
    else:
        response += '[请登录]'
    return response

```

#### 登出
登出账户的世纪操作就是把代表用户认证的cookie删除，这通过session.pop()方法实现
```python
from flask import session

@app.route('/logout')
def logout():
    session.pop('logged_in')
   return redirect(url_for('hello'))

```

### 完整代码
```python
from flask import Flask, request, make_response, redirect, url_for, session
from jinja2 import escape

app = Flask(__name__)

app.secret_key = 'adadanjkdhoaf29891'

@app.route('/hello')
def hello():
    # 从query string里获取name参数
    name = request.args.get('name')
    if name is None:
        name = request.cookies.get('name', '游客')
    # escape 防止XSS攻击
    response = 'hello %s' % escape(name)
    if session.get('logged_in'):
        response += '[已登录]'
    else:
        response += '[请登录]'
    return response


@app.route('/login/<name>')
def login(name):
    response = make_response(redirect(url_for('hello')))
    response.set_cookie('name', name)
    session['logged_in'] = True
    return response


@app.route('/logout')
def logout():
    if 'logged_in' in session:
        session.pop('logged_in')
    return redirect(url_for('hello'))


```

在浏览器分别访问
- `localhost:5000/`
- `localhost:5000/hello`
- `localhost:5000/login/baichen`
- `localhost:5000/logout`


