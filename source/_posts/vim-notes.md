---
title: vim笔记 
date: 2020-11-21 12:33:48
tags:
- vim
categories:
- vim
preview: 100
---

## 移动光标

gg 移动到第一行
G	移动到最后一行
0 到行首（第 1 列）
^ 到第一个非空白字符
$ 到行尾
<n>gg  移动到第n行
Ctrl-d 向下移动半页
Ctrl-u 向上移动半页



## 增删改查

### 增
a (append)
i (insert)
o (open a line)
以上三种都是在当前字符/行后插入

A	行尾插入
I	行头插入
O	向上新建一行

### 删 d delete
d$	删除当前字符到行末的内容
x	删除当前字符
dd	剪切当前行
di(	删除()中间的字符
dw	删除当前单词

### 改 c change
u	撤销
.	 重复
:%s/a/b   全局把a替换成b
:s/a/b	替换当前行第一个a为b
:s/a/b/g	替换当前行所有a为b
:n,$s/a/b	替换第n行到最后一行的a为b


ciw 	change inner word

- 比如(asdasd) 输入ci(就可以修改括号中间的内容
ctw	change to word
- 比如ct) 就修改 asdasd) 做括号左边的内容

### 查 f find

f<x>	在当前行查找第一个字符x，按;查找下一个，按,查找上一个
F<x>	反向查找

/word	全文查找单词 按n查找下一个，按N查找上一个
?word	反向查找

### 复制粘贴

y	复制
yy	复制当前行
p	粘贴
P	在上一行粘贴
### 格式

<opration> <motion>
d3h	向左删除三个字符
df:	删除当前行:前的内容
y$	复制当前字符到末尾

## 配置文件 .vimrc

在`~`文件夹下新建一个.vimrc文件

## visual 模式

在普通模式下按v进入可视模式
V	进入可视行模式
- 选中后输入:normal <command> 可以执行normal指令
- 选中后:normal A.png 在每行后面添加.png后缀
ctrl+v	进入可视块模式


## 分屏

## 插件

到[https://github.com/junegunn/vim-plug](https://github.com/junegunn/vim-plug)根据提示安装`vim-plug` 并且修改配置文件

## 其他
[重复技巧](https://blog.csdn.net/ii1245712564/article/details/46496347)