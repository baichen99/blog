---
title: flask06-数据库
date: 2018-11-24 15:31:00
tags:
- flask
- 数据库
- ORM
- Database
categories:
- flask学习笔记
preview: 100
---

### ORM
在web应用里使用原声sql语句操作数据库主要存在以下两类问题:

1. 手动编写sql语句乏味，而且会降低代码易读性。还会出现安全问题
2. 常见的开发模式时在开发时使用简单的SQLite，在部署时切换到MySQL等更健壮的DBMS。但是对于不同的DMBS，我们需要使用不同的python借口库，这让DBMS切换不那么容易。

----
尽管使用ORM可以避免sql注入问题，但你仍然需要对传入的查询参数进行验证。另外在执行原生SQL语句时也要注意避免使用字符串拼接或字符串格式化的方式传入参数。

----

使用ORM可以很大程度上结局这些问题：

1. 自动处理查询参数的转义，避免sql注入。
2. 为不同dbms提供统一接口，让切换工作变得简单。
3. ORM将python语言转换成dbms能够读懂的sql指令，能让我们利用python操控数据库。

####  ORM的三层映射关系
- 表 --> Python类
- 字段(列)  -->  类属性
- 记录(行)  -->  类实例

## 使用Flask-SQLAlchemy管理数据库
安装Flask-SQLAlchemy及其依赖
```
$ pipenv install flask-sqlalchemy
```

实例化Flask-SQLAlchemy提供的SQLAlchemy类，传入程序实例完成初始化，吧实例化扩展类的对象命名为db，这个db对象代表我们的数据库。

```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

db = SQLAlchemy(app)

```

### 连接数据库服务器
|DBMS|URI|
|:------|:--------:|
|MySQL|mysql://username:password@host/databasename|
|SQLite(Unix)|sqlite:////absolute/path/to/foo.db|
|SQLite(Windows)|sqlite:///absolute\\path\\to\\foo.db|
|SQLite(内存型)|sqlite:///或sqlite:///:memory|

在Flask-SQLAlchemy中，数据库的URI通过配置变量SQLALCHEMY_DATABASE_URI设置，默认为SQLite内存型数据库。
SQLite是基于文件的dbms，不需要设置数据库服务器，只需要制定数据库文件的绝对路径。使用app.root_path来定位数据库文件的绝对路径，并将数据库文件命名为data.py。

```
import os
...
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URL', 'sqlite:///' + 
    os.path.jpin(app.root_path, 'data.db'))
    
```
`os.getenv(key, default)` Return the value of the environment variable key if it exists, or default if it doesn’t.


安装并初始化Flask-SQLAlchemy后，启动程序会看到命令行有一段警告信息。这是因为Flask-SQLAlchemy建议你设置`SQLALCHEMY_TRACK_MODIFICATIONS`配置变量，这个配置决定是否追踪对象的修改，这用语Flask-SQLAlchemy的事件通知系统，如果没有特殊需求设置为False，来关闭警告信息：
```
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
```

### 定义数据库模型
用来映射到数据库表的python类被称为数据库模型(models)。一个数据库模型类对应数据库中的一个表，所有的模型类都需要继承Flask-SQLAlchemy提供的db.Model基类。
```Python
class Note(db.model):
    id = db.Colum(db.integer, primary_key=True)
    body = db.Colum(db.Text)

```
**SQLAlchemy常用字段类型**
![](https://s1.ax1x.com/2018/11/24/FFhmPe.png)
字段类型一般直接声明，如果要传入参数，也可以添加括号。

---

默认情况下，Flask-SQLAlchemy会根据模型类的名称生成一个名称，生成规则如下:
- 单个单词转换为小写
- 多个单词转换为小写并使用下划线分割

如果想自己指定表名称，可以通过定义__tablename__属性来实现。

**实例化字段类常用的字段参数**
![](https://s1.ax1x.com/2018/11/24/FFhsaT.png)

实例化字段类时，通过把参数`primary_key`设为True将其定义为主键。主键是每一条记录独一无二的标识，也是模型类中必须定义的字段。

### 创建数据库和表

创建模型类后，我们需要手动创建数据库和对应的表。这通过db对象的`create_all()`方法实现。
```
from app import db

db.create_all()

```
----
数据库和表一旦建立后，之后对模型的改动不会自动作用到实际的表中。比如在模型类中添加或删除字段，修改字段的名称和类型，这时再次调用`create_all()`方法也不会更新表结构。如果要使改动生效，最简单的方式是调用`db.drop_all()`方法删除数据库和表，然后再用`create_all()`方法创建。

----

### 数据库操作
SQLAlchemy使用*数据库会话/事务(transaction)*来管理数据库操作，Flask-SQLAlchemy自动帮我们创建绘画，可以通过`db.session`属性来获取。这里的会话对象和Flask中的session无关。

数据库中的会话代表一个临时储存去，你对数据库做的改动都在这里，只有对session对象调用`commit()`方法时，改动才会被提交到数据库。会话也支持回滚操作，当调用`rollback()`方法是，添加到会话中且未提交的改动都将被撤销。

### CURD

#### Create
添加一条新记录到数据库主要分为三步
1. 实例化模型类作为一条记录
2. 添加新创建的记录到数据库会话
3. 提交数据库会话
```Python
from app import db, Note

note1 = Note(body="text one")
note2 = Note(body="text two")
note3 = Note(body="text three")
db.session.add(note1)
db.session.add(note2)
db.session.add(note3)

db.session.commit()

```
Note类继承自db.Model类，db.Model基类会为Note类提供一个构造函数，接收匹配类属性名称的参数值，并赋值给对应的类属性。

### Read
使用模型的query属性**附加调用**各种过滤方法及查询方法可以从数据库中取出数据。
一般来说，一个完整的查询模式遵循下面的模式。

<模型类>.query.<过滤方法>.<查询方法>


常用的SQLAlchemy查询方法

|查询方法|说明|
|:-----|--------|
|all()|返回包含所有查询记录的列表|
|first()|返回查询的第一条记录，如果未找到，则返回None|
|one()|返回第一条记录，若记录数量!=1则抛出错误|
|get(indent)|传入主键值作为参数，返回指定主键值的记录，如果未找到，则返回None|
|count()|返回查询结果的数量|
|one_or_none()|如果结果数量不唯一，则返回None|
|first_or_404|返回第一条，如果未找到则返回404错误响应|
|get_or_404(indent)|传入主键值作为参数，返回对应主键值的记录，如果未找到，则返回404错误响应|
|paginate|返回一个Pagination对象，可以对记录进行分页处理|
|with_parent(instance)|传入模型实例作为参数，返回和这个实例相关联的对象|

SQLAlchemy还提供了许多过滤方法，可以获取更精确的查询。因为对Query对象调用过滤方法将返回一个更精确的Query对象，使用过滤器可以叠加使用。

常用的SQLAlchemy过滤方法

|查询过滤器|说明|
|:-------|:---------|
|filter()|使用指定的规则过滤记录，返回新产生的查询对象|
|filter_by()|使用指定规则过滤记录，返回新的查询对象|
|order_by|根据指定条件对记录进行排序，返回新产生的查询对象|
|limit(limit)|根据指定的值限制原查询返回的记录数量，返回新产生的查询对象|
|group_by()|根据指定的值对记录进行分组，返回新产生的查询对象|
|offset(offset)|使用指定的值偏移原查询结果，返回新产生的查询对象|

`Note.query.filter(Note.body=='note one').first()`
其他常用查询操作符：
**LIKE**
```
filter(Note.body.like('%foo%'))
```

**IN**
```
filter(Note.body.in(['foo', 'bar', 'baz']))
```

**NOT IN**
```
filter(~Note.body.in(['foo', 'bar', 'baz']))
```

**AND**
```
from sqlalchemy import and_
# 使用and_()
filter(and_(Note.body=='bar', Note.title=='foo'))

# 使用逗号分隔
filter(Note.body=='bar', Note.title=='foo')

# 叠加使用过滤方法
filter(...).filter(...)

```

**OR**
```
from sqlalchemy import or_
filter(or_(Note.body=='foo', Note.title=='bar'))
```

### Upate
直接赋值给模型类的字段属性就可以改变字段值，然后调用`commit()`方法提交会话即可。

### delete
和添加记录类似，不过要用`delete()`方法，最后调用`commit()`方法提交修改。
```
>>> note = Note.query(2)  # 主键为2
>>> db.session.delete(note)
>>> db.commit()
```

## 在视图函数中操作数据库
### Create
```
@app.route('/new', methods=['POST', 'GET'])
def new_note():
    form = NewNoteForm()
    if form.validate_on_submit():
        body = form.body.data
        note = Note(body=body)
        db.session.add(note)
        db.session.commit()
        flash('your note is saved')
        return redirect(url_for('index'))
    return render_template('new_note.html', form=form)
    
```

### Read
```
@app.route('/')
def index():
    form.DeleteForm()
    notes = Note.query.all()
    return render_template('index.html', notes=notes, form=form)

```
### Update
和create相似
```
@app.route('/edit/<int:note_id>', methods=['GET', 'POST'])
def edit_note(note_id):
    form = EditNoteForm()
    note = Note.query.get(note_id)
    if form.validate_on_submit():
        note.body = form.body.data
        db.session.commit()
        flash('note is updated')
        return redirect(url_for('index'))
    form.body.data = note.body
    return render_template('edit_note.html', form)
```

### Delete
不能添加一个指向删除视图的链接，因为有CSRF攻击的风险，像删除这类修改数据的操作绝对不能通过get请求实现，**正确的做法**是为删除操作创建一个表单。如下
```python
class DeleteNoteForm(FlaskForm):
    submit = SubmitField('Delete')
```

```
@app.route('/delete/<int:note_id>', methods=['POST'])
def delete_note(note_id):
    form = DeleteNoteForm()
    if form.validate_on_submit():   # 主要是验证CSRF令牌，要在模板中手动渲染form.csrf_token
        note = Note.query.get(note_id)
        db.session.delete(note)
        db.session.commit()
        flash('note is deleted')
    else:
        abort(400)
    return redirect(url_for('index'))

```

## 定义关系
### 一对多
```
class Author(db.Model):
    id
    name
    phone

class Article(db.Model):
    id
    title
    body
```
目的是在Author类中添加一个articles属性，当对特定的Author对象调用articles属性会返回相关Article对象。
#### 一、定义外键
外键是在表A存储表B的*主键值*，以便和表B建立联系的关系字段。因为怪键只能存储单一数据（标量），使用外键总是在“多”的一侧定义，多篇文章属于一个作者，所以我们需要在文章添加外键，存储作者的主键值以指向对应的作者。
```
class Article(db.Model):
    ...
    author_id = db.Colum(db.Interger, db.ForeignKey('author.id'))
```
外键字段的命名没有限制，但传入ForeignKey类的参数`author.id`中，author指的是表名，id是字段名。默认为类的名称的小写形式，多个单词通过下划线分割。可以通过`__tablename__`属性指定。

#### 二、定义关系属性
定义关系的第二部是使用关系函数定义关系属性。**关系属性在关系的出发侧定义**
```
class Author(db.Model):
    ...
    articles = db.relationship('Aticle')
```
这个属性并没有使用Column类来声明为列，而是使用了`db.relationship()`定义为关系属性。因为这个关系属性返回多条记录，我们称之为*集合关系属性*。
relationship()函数的第一个参数为关系另一侧模型的名称，他会告诉SQLAlchemy将Author类与Article类建立关系。当这个关系被调用时，SQLAlchemy会找到关系另一侧的外键字段(author_id)，然后反响查询article表中所有author_id值为当前主键值(author.id)的记录。
```
>>> foo = Author(name='foo')
>>> spam = Aticle(title='spam')
>>> ham = Article(title='ham')
>>> db.session.add(foo)
>>> db.session.add(spam)
>>> db.session.add(ham)

```
### 三、建立关系
有两种方式，第一种是为外键赋值。
```
>>> spam.author_id = 1
>>> db.session.commit()
```
将spam的author_id设为1，会和id为1的Author建立关系。

另一种方式是通过操作关系属性，将关系属性赋值给实际的对象即可建立关系。
```
>>> foo.articles.append(spam)
>>> foo.articles.append(ham)
>>> db.session.commit()
```
使用`remove()`方法可以解除关系，`pop()`方法可以与关系属性列表的最后一个对象解除关系并返回该对象。

常用的SQLAlchemy关系函数参数

|参数名|说明|
|:--------|:-----|
|back_populates|定义反向引用，用于建立双向关系，在关系的另一侧也必须现实定义关系属性|
|backref|添加反向引用，自动在另一侧建立关系属性|
|lazy|指定如何加载相关记录|
|useklist|指定是否使用列表形式加载记录|
|cascade|设置级联操作|
|order_by|指定家在相关记录时的排序方式|
|secondary|在多对多关系中指定关联表|
|primaryjoin|指定多对多关系中的一级联结条件|
|secondaryjoin|指定多对多关系中的二级联结条件|

常用的SQLAlchemy关系记录加载方式(lazy参数可选值)

|关系加载方式|说明|
|:----|:-------|
|select|在必要时一次性加载疾苦，返回包含记录的列表，等同于lazy=True|
|joined|和父查询一样加载记录，但使用联结，等同于lazy=False|
|immediate|一旦父查询加载就加载|
|subquery|类似joined，不过使用自查询|
|dymamic|不知加载记录，而是返回一个包含相关记录的query对象|

### 建立双向关系
两侧都添加关系属性获取对方记录的关系为双向关系。双向关系的建立很简单，通过在关系的另一侧也创建一个relationship()函数，我们就可以在两个表之间建立双向关系。
















