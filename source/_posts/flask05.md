---
title: flask05-表单
date: 2018-11-22 13:17:00
tags:
- flask
- 表单
- forms
categories:
- flask学习笔记
preview: 100
---

## 表单

## html表单
在html表单中，`<input>`标签表示各种输入字段，`<label>`标签则用文字来定义字段的标签文字。

```HTML
<form mehtod="post">
    <label for="username">Username</label></br>
    <input type="text" name="username" placeholder="请输入"></br>
    <label for="password">Password</label></br>
    <input type="password" name="password" placeholder="1414124"></br>
    <input type="checkbox" name="remenber" id="remember" checked></br>
    <label for="remenber">Remember me</label></br>
    <input type="submit" name="submit" value="login">
</form>
```

## WTForms
WTForms支持在python中使用**类**来定义表单，然后通过类定义生成对应的html代码，这种方式方面，易于重用，所以一般不会再模板中直接使用html编写表单。

## 使用Flask-WTF处理表单
Flask-WTF集成了WTForms使用它可以在flask中更方便地使用WTFormss。Flask-WTF将表单数据解析，CSRF保护，文件上传等功能与flask集成，还增加了reCAPTCHA支持。

安装Flask-WTF及其依赖
`$ pipenv install flask-wtf`

Flask-WTF默认为每个表单启用CSRF保护，他会为我们自动生成和验证CSRF令牌。默认情况使用程序密钥来对令牌进行签名，配置键`WTF_CSRF_ENABLED`来设置是否开启CSRF保护，默认为True。flask-wtf会自动在实例化表单类时添加一个包含CSRF令牌值的*隐藏字段*，字段名为`cstf_token`

```
app.secret_key = 'secret string'
```

### 定义WTForms表单类
当使用WTForms创建表单时，表单由python来表示，这个类继承从WTForms导入Form基类。一个表单由若干输入字段(Field)组成，字段由表单类的类属性表示。下面自定义了一个LoginForm类。
```python
from wtforms import Form, StringField, PasswordField, BooleanField, SubmitField
from wtfroms.validators import DataRequired, Length
class LoginForm(Form):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired(), Length(8, 128)])
    remember = BooleanField('Remember me')
    submit = SubmitField('Login')
```
每隔字段属性通过实例化WTForms提供的字段表示，字段属性的名称将作为对应input元素的name属性和id属性。</br>
字段名称大小写敏感，不能以下划线和validate开头。

### 常用WTForms字段
![](https://s1.ax1x.com/2018/11/22/FPNu9J.png)

### 实例化字段类常用参数
|参数|说明|
|:-----|:--------------:|
|label|字段标签<label>的值，也就是渲染后显示在输入字段前的文字|
|render_kw|一个字典，用来设置对应input标签的属性|
|valiadtors|一个列表，包含一系列验证器，会在表单提交后被逐一调用验证表单数据|
|default|字符串或可调用对象，用来为表单字段设置默认值|
### 常用WTForms验证器

![](https://s1.ax1x.com/2018/11/22/FPNQj1.png)

验证器的第一个参数一般为促物体时消息，可以使用message参数传入错误消息来覆盖内置消息。
```
name = StringField('Your name', validators=[DataRequired(message=u"名字不能为空")])
```
### 使用Flask-WTF
使用Flask-WTF定义表单时，**仍然使用WTForms提供的字段类和验证器**，只不过表单类药继承Flask-WTF提供的FlaskForm类。
```
from flask-wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtfroms.validators import DataRequired, Length
class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired(), Length(8, 128)])
    remember = BooleanField('Remember me')
    submit = SubmitField('Login')
```

## 输出html代码
实例化表单类，然后将实例属性转换成字符串或直接调用获取表单字段对应的html代码
```
>>> form = LoginForm()
>>> form.username()
u'<input id="username" name="username" type="text" value="">'
```
字段的label元素的html代码可以通过`form.字段名.label`的形式获取

### 为字段添加额外属性
#### 使用render_kw属性
```
username = StringField('Username', render_kw={'placeholder':'Your name'})
```
#### 在调用字段时传入
```
form.username(style='width: 200px;', class_='bar')
```
用class_代替class，渲染后的input标签会获取正确的class属性。

## 在模板中渲染表单
```
from forms import LoginForm

@app.route('/basic')
def basic():
    form = LoginForm()
    return render_template('login.html', form=form)
```

在模板中通过调用属性获取字段对应的html代码

```
<form method="post">
    {{ form.csrf_token }}
    {{ form.username.label }}  {{ from.username }}</br>
    {{ form.password.label }}  {{ form.password }}</br>
    {{ form.remember }}  {{form.remember.label}}</br>
    {{ form.submit }}</br>
    
</form>
```
csrf_token在提交表单后会被自动验证，为了确保表单通过验证，必须在表单中手动渲染这个字段。

## 处理表单数据
1. 解析请求，获取表单数据
2. 对数据进行必要的转换
3. 验证数据和CSRF令牌
4. 如果验证未通过则生成错误消息
5. 如果通过验证则把数据保存到数据库或做进一步处理

### 提交表单
<form>标签的提交行为主要由三个属性控制。

|属性|默认值|说明|
|:-----|:------:|:------|
|action|当前url|表单提交时发送请求的url|
|method|get|提交方法，仅支持get和post|
|enctype|application/x-www-form-urlencoded|表单编码类型|

### 验证表单数据
#### wtforms验证机制
实例化表单类时传入表单数据，然后对表单实例调用`validate()`方法。这会对每个字段调用相应的验证器，返回验证结果的布尔值。如果验证失败，就把错误消息储存到表单实例的errors属性对应的字典中。

---

注意：使用post方法提交的表单数据会被解析成一个字典，可以通过`request.form`获取。使用get方法提交的数据同样被解析成字典，通过`request.args`获取。

---

#### 在视图函数中验证表单
``` 
if request.method=='POST' and form.validate(): 
```

Flask-WTF提供的`validate_on_submit()`合并了这两个操作

``` 
if form.validate_on_submit(): 
```

通过form.字段名.date获取对应字段的数据。

#### Post/Redirect/Get
在浏览器中，刷新/重载时的默认行为时发送上一个请求，如果上一个请求时post，那么就会弹出一个窗口，询问用户是否再次提交此表单，为了避免这个提示，我们尽量不要让post请求作为最后一个请求，所以我们在处理表单后返回一个重定向响应，让浏览器发送一个get请求到相应url。
```
...
@app.route('/basic', methods=['GET', 'POST'])
def basic():
    form = LoginForm()
    if form.validate_on_submit():
        username = form.username.data
        flash("Welcome Home %s", % username)
        return redirect(url_for('index'))
    return render_template('basic.html', form=form)
```

### 在模板中渲染错误消息
我们一般会通过字段名来获取**对应字段的错误消息列表**，即`form.字段名.errors`。


## 自定义验证器
### 行内验证器
当表单类中包含以`validate_字段属性名`形式命名的方法时，在验证字段数据时会同时调用这个方法来验证对应的字段。验证方法接收两个位置参数，以此为form和field，分别是表单类实例和字段对象。验证出错跑出从`wtforms.validators`模块导入的`ValidatorError`异常。

注：不需要传入参数时使用

```
from wtforms import IntegetField, SubmitField
from wtforms.validators import ValidationError

class FourtyTwoForm(FlaskForm):
    answer = IntegerField('The Number')
    submit = SubmitField()
    
    def validate_answer(form, field):
        if field.data != 42:
            raise VlidationError("Must be 42")
```

### 全局验证器
如果想定义一个可重用的验证器，可以通过一个函数实现。
```
def is_42(message=None):
    if message is None:
        message = 'Must be 42'
    def _is_42(form, field):
        if field.data != 42:
            raise ValidationError(message)
    
    return _is_42
            
```

注意：自定义验证器要返回可调用对象






