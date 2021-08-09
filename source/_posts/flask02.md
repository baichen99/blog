---
title: flask02-启动开发服务器
date: 2018-09-22 12:58:39
tags:
- flask
- server
categories:
- flask学习笔记
preview: 100
---


flask通过依赖包Click安装了一个CLI(命令行交互界面)系统, 可以通过flask命令执行内置命令
使用`flask run`来启动内置的开发服务器
<!-- more -->
使用命令之前确保激活了虚拟环境

如果执行`flask run`命令后显示(command not found)或其他错误，可以尝试使用`python -m flask run`启动服务器

旧的启动开发服务器的方式是使用`app.run()`方法，目前已不推荐使用

### 自动发现程序实例
在执行flask run 命令之前我们需要提供程序实例所在模块的位置，我们之所以可以直接运行是因为flask会自动探测程序实例，自动探测存在下面的规则
1. 从当前目录寻找app.py或wsgi.py的模块，并从中寻找名为app.py或application的程序实例
2. 从环境变量FLASK_APP对应的值寻找名为app.py或application的程序实例

如果程序主模块是其他名称，比如`hello.py` ，需要设置环境变量FLASK_APP
在macos/linux下
`$ export FLASK_APP=hello`

在windows下
`> set FLASK_APP=hello`

### 管理环境变量
环境变量在创建新命令行和重启电脑之后就清除了，为了避免频繁设置环境变量，可以使用python-dotenv管理项目的环境变量
安装: `pipenv install python-dotenv`
1. 在项目根目录创建两个文件`.flask`和`.env`，环境变量使用键值对定义，每行一个，#为注释
```python
SOME_VAR=1
#这是注释
FOO = 3

```

### 更多启动选项
#### 使服务器对外部可见
`$ flask run --host=0.0.0.0`
#### 设置端口
`4 flask run --port=8000`

