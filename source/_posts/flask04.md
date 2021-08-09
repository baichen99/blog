---
title: flask04-模板
date: 2018-11-05 19:42:03
tags:
- 模板
- flask
categories:
- flask学习笔记
preview: 100
---

flask使用的默认模板引擎是`jinja2`



## 模板语法
### 三种界定符
1. 语句，比如if、for

{% raw %}
{% ... %} 
{% endraw%}
2. 表达式比如字符串，变量，函数调用

{% raw %}
{{ ... }}
{% endraw%}

3. 注释

{% raw %}
{# ... #}
{% endraw %}

在jinja2使用`.`来获取变量的属性，比如user字典中的username键值通过`.`来获取j，即`user.username`, 等同于`user[username]`

1. jinja2并不支持所有Python所有语法，使用处于效率和代码组织等方面考虑**应该适度使用模板，仅把和输出控制有关的逻辑操作放到模板中**
2. Jinja2提供了多种控制结构来控制模板的输出，其中`for` 和 `if` 是最常用的两种，在语句结束时要添加 **结束标签**


``` 
{% if user.bio %}
<i> {{ user.bio }} </i>
{% else %}
<i> This user has not provided a bio. </i>
{% endif %}
```


```Jinjia
<ul>
{% for movie in movies %}
<li> {{ movie.name }} -- {{ movie.year }} </li>
</ul>
```

### for循环特殊变量
|变量名|说明|
|:------------------:|:-----------:|
| loop.index | 当前迭代数（从1开始计数） |
| loop.index() | 当前迭代书（从0开始计数） |
| loop.revindex | 当前反向迭代数（从1开始计数） |
| loop.revindex() | 当前反向迭代数（从0开始计数） |
| loop.first | 如果时第一个元素则返回True |
| loop.last | 如果时最后一个元素则返回True |
| loop.previtem | 上一迭代条目 |
| loop.nextitem | 下一迭代条目 |
| loop.length | 序列包含的元素数量 |

### 渲染模板
渲染一个模板就是执行模板中的代码，并传入所有在模板中使用的变量，渲染后的结果就是我们要返回客户端的HTML响应，在视图函数中我们并不直接使用Jinjia2提供的函数，而是使用Flask提供的渲染函数`render_template()`

```Python
from flask import Flask, render_template
...

@app.route('/watchlist')
def watchlist():
return render_template("watchlist.html", user=user, movies=movies)
```

在`render_template()`函数中，第一个参数时模板的文件名, Flask会在程序根目录下的templates文件夹里寻找模板文件，所以这里传入的路径时相对于templates根目录的。以关键字的形式传入了模板中使用的变量值，其他类型的变量通过相同的方式传入，传入的变量值可以时字符串、列表和字典，也可以时函数、类和实例

如果想传入函数，在模板调用，那么需要传入函数对象本身，而不是函数调用（函数的返回值），使用仅写出函数名即可，传入模板后，在函数名称后面添加括号以调用，以可以在括号中传入参数

## 模板辅助工具
### 上下文

模板上下文包含了很多变量，其中包裹我们调用`render_template()`时传入的变量，除了在渲染时传入变量，也可以在模板中定义d变量，使用{% raw %}{% %}{% endraw %}标签

```
{% set navigation =[ ('/', 'Home'), ('/about', 'About') ] %}
```

也可以将一部分模板数据定义为变量使用 `set`和`endset`来声明开始和结束
```
{% set navigation %}
<li><a href="/">Home</li>
<li><a href="/about">About</li>
{% endset %}
```

#### 内置上下文变量
|变量|说明|
|:------------------:|:-----------:|
| config | 当前的配置对象 |
| request | 当前的请求对象，在已激活的请求环境下可用 |
| session | 当前的会话对象，在已激活的请求环境下可用 | 
| g | 与请求绑定的全局变量，在已激活的请求环境下可用 |
#### 自定义上下文
如果很多个模板都需要使用同一变量，那么可以设置一个模板全局变量避免在多个视图函数中重复传入。flask提供了一个`app.context_processor`装饰器，可以用来注册模板上下文处理函数，它可以帮我们完成统一传入变量的工作。模板上下文处理函数需要返回一个**包含变量键值对的字典**

```Python
@app.context_processor()
def inject_foo():
foo = "I am foo"
return dict(foo=foo)
```

当我们调用`render_template()`函数渲染任意一个模板时，所有使用`app.context_processor`装饰器注册注册的模板上下文处理函数（包括flask内置的上下文处理函数）都会被执行，这些函数的返回值会被添加到模板中，因此我们可以直接在模板中使用`foo`变量

除了使用`app.context_processor`装饰器，也可以直接将其作为方法调用，传入模板上下文处理函数
```Python
...
def inject_foo():
foo = "I am foo"
return dict(foo=foo)

app.context_processor(inject_foo)

```

## 全局对象
全局对象是指在所有的模板中都可以直接使用的对象
### 内置全局函数
Jinjia2在模板中默认提供了一些全局函数，常用的三个函数

|函数|说明|
|:-------------|:-----|
| range([start, ] stop[, step]) | 和python中range()用法相同 |
| lipsum(n=5, html=True, min=20, max=100) | 生成随机文本，可以在测试时调用来填充页面，默认生成5段html文本，每段包含20～100个单词 |
| dict(**items) | 和python中dict()用法相同 |

flask在模板中内置了两个全局函数

|函数|说明|
|:------------------|:-----------|
| url_for() | 用于生成url|
| get_flashed_messages() | 获取flash消息 |

url_for()传入视图端点
### 自定义全局函数
我们可以使用`app.template_global`装饰器直接将函数注册为模板全局函数。默认使用函数名称传入模板，在`app.template_global()`装饰器中使用name参数可以指定一个自定义名称。

## 过滤器
在jinja2中，过滤器是一些可以用来修改和过滤变量值的特殊函数，过滤器和变量之间用`|`隔开，需要参数的过滤器可以用括号进行传递

#### 例子1
```{{name|title}}``` 这会将name变量的值标题化，相当于在Python中调用`name.title()`
#### 例子2
```{movies|length}```可以获取watchlist列表的长度，类似在Python中调用`len(movies)`</br>

另一种方法时将过滤器作用于一部分模板数据，使用`filter`和`endfilter`标签声明开始和结束
```jinjia2
{%filter upper%}
    This text will be uppercase.
{%endfilter%}
```
### 内置过滤器
|过滤器|说明|
|:------------------|:-----------|
| default(value, default_value=u'', boolean=False) | 设置默认值，默认值作为参数传入，别名为d|
|escape(s)|转移html文本，别名为e|
|first(seq)|返回序列的第一个元素|
|last(seq)|返回序列的最后一个元素|
|length(object)|返回变量的长度|
|random(seq)|返回序列中的随机元素|
|safe(value)|将变量值标记为安全，避免转义|
更多过滤器请访问[http://jinja.procc.org/docs/2.10/templates/#builtin-filters](http://jinja.procc.org/docs/2.10/templates/#builtin-filters)

在使用过滤器时，列表中过滤器函数的第一个参数表示被过滤的变量值(value)或字符串(s)，即竖线符号左侧的值，其他参数可以通过添加括号传入，另外过滤器可以叠加使用

```
<h1>Hello, {{name|default('陌生人')|title}}!</h1>
```

### 自定义过滤器
使用`app.template_filter()`装饰器可以注册自定义过滤器

```

from flask import Markup

@app.template_filter()
def musical(s):
    return s+Markup(' &#9835') # 在变量字符后添加一个音符图标，用markup将其标记为安全字符
    
```
和注册全局函数类似，可以在app.template_filter()中使用name关键字设置过滤器的名称，默认为函数名称。   </br>
过滤器需要接受被处理的值为输入，返回处理后的值。

## 模板组织结构
### 局部模板
在web程序中，我们通常会为每一页面编写一个独立的模板，比如主页模板，设置模板，这些模板可以**直接在试图函数中渲染并作为html响应主体**</br>
除了这类模板我们还会用到另一类非独立模板，被称为**局部模板或次模板**，因为他们仅包含部分代码，使用不会再视图函数中直接渲染它，而是插入到其他独立模板中。</br>
为了避免重复，当多个独立模板中都会使用同一块html代码时，我们可以把这部分代码抽离出来，存储在局部模板中。比如多个页面都要在顶部显示一个提示条，这个横幅抗议定义在局部模板`_banner.html`中。</br>
使用
```
{% inlcude '_banner.html' %}
```
来插入一个局部模板，这会吧局部模板的全部内容插在此标签的位置。
### 宏
宏类似于python里的函数。使用宏可以把一部分模板代码封装到宏里，使用传递的参数来构建内容，最后返回构建后的内容。功能上和局部模板相似，都是为了方便代码块的重用。</br>
在创建宏时使用`macro`和`endmacro`标签声明宏的开始和结束，在开始标签中定义宏的名称和接收的参数

```
{% macro qux(amount=1) %}
    {% if amount == 1 %}
        i am qux
    {% elif amount >1 %}
        we are quxs
    {% endif %}
{% endmacro %}

```
使用示例

{% raw %}
{% from 'macros.html' import qux %}</br>
...</br>
{{ qux(amount=5) }}</br>
{% endraw %}

### 模板继承

当子模板继承基模板后，子模板会包含基模板的内容和结构，为了让子模板能插入或覆盖内容到基模板中，我们需要在**基模板**中定义block。在子模板中使用{% raw %}{% extend 'base.html' %}{%endraw%}来声明扩展基模板。
*extend必须是子模板的第一个标签！*

#### 覆盖内容
当在子模板里创建同名的块时，会用子块内容覆盖父块内容
#### 追加内容
如果想向基模板的块中追加内容，需要用`super()`函数声明

{% raw %}
{% block styles %}</br>
{{ super() }}</br>
<style></br>
    .foo{color:red}</br>
</style></br>
{% endblock %}</br>
{% endraw %}


## 加载静态文件
Flask中默认需要将静态文件存储在与主脚本（包含程序实例的脚本）同级目录的static文件夹中， 如果想在其他文件夹中储存静态文件，可以在在实例化Flask类是使用`static_folder`参数指定，静态文件的url路径中的static也会自动随着文件夹名称变化。</br>
为了在html文件中引用静态文件，需要用`url_for()`函数获取静态文件url。flask内置了用于获取静态文件的师徒函数，端点值为static，它的默认url规则为`static/<path:filename>`，url变量filename是相对于static文件夹根目录的文件路径。

### 添加favicon（网站头像）
在static文件夹下放入favicon文件，命名为favicon.ico，然后再html页面中声明favicon路径
{% raw %}
<link rel="icon" type="image/x-icon" href={{url_for('static', filename='favicon.ico')}}>
{% endraw %}

## 消息闪现
flask提供的`flash()`函数发送消息储存在session中，在模板中使用全局函数`get_flashed_messages()`获取消息，并将其显示出来。 </br>
通过flash()函数发送的消息储存在session中，所以我们需要为程序设置密钥，可以通过`app.secret_key`属性或者配置环境变量`SECRET_KEY`设置

示例
{% raw %}
from flask import Flask, render_template, flash</br>

app = Flask(__name__)</br>
app.secret_key = 'secret string'</br>

@app.route('/flash')</br>
def just_flash():</br>
flash('i am flash')</br>
return redirect(url_for('index'))</br>
{% endraw %}

在templates中的代码如下

{% raw %}
<main></br>
    {% for message in get_flashed_messages() %}</br>
        <div class="alert"> {{message}} </div></br>
    {% endfor %}</br>
    {% block content %} {% endblock %}</br>
</main></br>
{% endraw%}

## 自定义错误页面

```
from flask import Flask, render_template
...
@app.errorhandler(404)
def page_not_found(e):
    return render_template('errors/404.html'), 404   
```
错误处理函数接收异常对象为参数，异常对象提供了以下常用属性
```
code  状态码
name    原因短语
description 错误描述
```
