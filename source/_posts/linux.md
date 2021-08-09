---
title: linux常见问题
date: 2020-08-17 14:53:11
tags:
- linux
categories:
- linux
---

## ps -elf

-e 显示所有进程

-f full format 完整格式

-l long formart 



UID： 说明该程序被谁拥有
PID：就是指该程序的 ID
PPID： 就是指该程序父级程序的 ID
C： 指的是 CPU 使用的百分比
STIME： 程序的启动时间
TTY： 指的是登录终端
TIME : 指程序使用掉 CPU 的时间
CMD： 下达的指令



## 软硬链接 ln

软链接：ln -s target linkname
硬链接：ln target linkname

硬链接 不占用空间，源文件和硬链接实际上是同一个文件，当两个文件都删除时，这个文件才会被删除，不能对目录进行链接

软链接相当于一个快捷方式，指向源文件，修改一个文件另一个文件也会跟着改变，但是删除源文件后，软链接就会失效，可以对目录进行链接

## 复制 cp [-r]

cp src target

cp -r src target  // 复制文件夹

## 删除 rm [-r]

rm -r dir // 递归删除

##chmod

https://blog.csdn.net/pythonw/article/details/80263428

修改文件权限

chomod 用户 操作符 权限 filename

用户有 u(文件所有者), g(文件所隶属的用户组), o(其他用户), a(全部用户)

权限分为w, r, x 分别是只读 只写 可执行



7 = 4 + 2 + 1    读写运行权限

5 = 4 + 1       读和运行权限

4 = 4          只读权限



`chmod a=rwx main.go` 给所有用户读写执行main.go的权限

`chmod 777 main.go ` 三个7分别给文件所有者、群组用户、其他用户权限




## grep

匹配正则表达式

`ps -ef | grep pid`查找特定进程



## 查看命令历史 history

hitory



## 查看磁盘使用状况

`df -hl`



## 查看/修改主机名2

`hostname`

显示： ecs-sn3-medium-2-linux-20200107151549

修改：

```
$ vim /etc/sysconfig/network

NETWORKING=yes
HOSTNAME=centos6.5-1
```

##进程状态

running 可执行状态，只有在该状态的进程才可能在CPU上运行。

sleeping 中断 在等待某个条件的形成或接收到信号

stopped

zombie  进程已终止，但进程描述还在，等到父进程调用wait后释放



## top

性能分析工具，能够实时显示系统中各个进程的资源占用状况

shift+f可以设置排序, 显示字段等选项



##  2> /dev/null 
[shell程序中 2> /dev/null 代表什么意思？](https://www.zhihu.com/question/53295083)



## 环境变量

如果要新增一条环境变量`/home/pi/.local/bin`可以使用`export PATH=$PATH:/home/pi/.local/bin`  其中`$PATH`是当前环境变量，各个环境由:分割，所以这里其实是把以前的环境变量和新增的拼接后重新赋值

[Linux环境变量配置全攻略 - 悠悠i - 博客园 (cnblogs.com)](https://www.cnblogs.com/youyoui/p/10680329.html)



## >, <, >>, << 重定向符

[Shell 输入/输出重定向 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-shell-io-redirections.html)





