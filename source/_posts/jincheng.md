---
title: 进程
date: 2018-06-15 20:52:45
categories:
- python
tags:
- 进程
- python
preview: 100
---


### 单核如何实现多任务? 
依次轮流执行, 看上去像是同时进行

### 并发与并行?
**并发**是看上去一起执行,任务数大于CPU核心数  **并行**是真正的一起执行, 任务数小于CPU核心数
<!-- more -->
### 线程与进程
. 线程：是程序执行流的最小单元，是系统独立调度和分配CPU（独立运行）的基本单位
. 进程：是资源分配的基本单位。一个进程包括多个线程。 对于操作系统而言, 一个任务就是一个进程

**区别**：
1. 线程与资源分配无关，它属于某一个进程，并与进程内的其他线程一起共享进程的资源。

2. 每个进程都有自己一套独立的资源（数据），供其内的所有线程共享。

3. 不论是大小，开销线程要更“轻量级”

4. 一个进程内的线程通信比进程之间的通信更快速，有效。（因为共享变量）

### 实现多任务的方式
1. 多进程模式
2. 多线程模式
3. 协程模式
4. 多进程+多线程模式

### 单任务现象
``` Python
from time import sleep

def run():
while True:
    print("hello")
    sleep(1)
if __name__ == '__main__':
    while True:
        print('bye')
        sleep(1)
    run()
```
**这样不会执行run(), 只有上面的while循环结束后才可以执行**

### 实现多任务
**multiprocessing库** --跨平台的多进程模块
``` Python
from multiprocessing import Process
from time import sleep
import os

def run():
    while True:
        print("hello---%s"%os.getpid()) # os.getpid()获取当前进程id号, os.getppid()获取当前进程的父进程的id号
        sleep(1)
if __name__ == '__main__':
    print("主(父)进程启动---%s"%os.getpid())
    # 创建子进程
    p = Process(target=run)   # 创建进程是为了执行函数, 需要传入两个参数 target=函数和arg=() 传入的参数必须元组, 如果元组只有一个元素后面必须加个逗号, 这样才构成元组
    # 启动进程
    p.start()
    while True:
        print('bye')
        sleep(1)
    run() 
```

### 父子进程的先后顺序
```python
import os
from multiprocessing import Process
from time import sleep

def run(str):
    print("子进程启动")
    sleep(3)
    print("子进程结束")

if __name__ == '__main__':
    print("父进程启动")
    p = Process(target=run, args=("nice",))
    p.start()

    print("父进程结束")
```
**运行结果**
```
父进程启动
父进程结束
子进程启动
子进程结束
```
**父进程的结束不影响子进程, 怎么让父进程等大子进程结束再执行父进程?**
```python
import os
from multiprocessing import Process
from time import sleep

def run(str):
    print("子进程启动")
    sleep(3)
    print("子进程结束")

if __name__ == '__main__':
    print("父进程启动")
    p = Process(target=run, args=("nice",))
    p.start()
    
    p.join() # 等待进程结束
    print("父进程结束")
```
### 全局变量在多个进程中不能共享
```python
from multiprocessing import Process
from time import sleep

num = 100


def run():
    print("子进程开始")
    global num
    num += 1
    print("子进程结束--%d" % num)


if __name__ == '__main__':
    print("父进程开始")

    # 创建子进程
    p = Process(target=run)
    p.start()
    p.join() # 等待进程结束

    print("父进程结束--%s"%num)
```
执行结果
```
 父进程开始
 子进程开始
 子进程结束--101
 父进程结束--100
```
- 在子进程中修改全局变量对父进程全局变量没有影响, 因为在创建子进程时对全局变量做了一个备份, 父进程和子进程的变量独立
- 每个进程都有自己的代码段, 堆栈段, 数据段

如果再创建一个子进程
```python
# 创建子进程
p2 = Process(target=run)
p2.start()
p2.join()  # 等待进程结束
```
变量依然独立

### 启动大量子进程
```python
from multiprocessing import Pool
import os, time, random

def run(name):
    print("子进程%s启动--%s" % (name, os.getpid()))
    start = time.time()
    time.sleep(random.choice[1, 2, 3])
    end = time.time()
    print("子进程%s结束--%s--耗时%.2f" % (name, os.getpid(), end - start))

if __name__ == '__main__':
    print("父进程启动")
    # 创建多个进程使用进程池
    pp = Pool(2) # 表示可以同时执行的进程数, 默认值是CPU核心数
    for i in range(5):
        # 创建进程, 放入进程池, 同一管理
        pp.apply_async(run, args=(i,))
    # 使用进程池时,在调用之前必须先调用close(), 调用close之后就不能添加新的进程了 
    pp.close() 
    # 等待进程池中所有子进程结束再去执行父进程
    pp.join()
    print("父进程结束")
```
### 进程间通信
进程间通过队列通信, 队列相当于一个中间人的角色
Process之间肯定是需要通信的，操作系统提供了很多机制来实现进程间的通信。Python的multiprocessing模块包装了底层的机制，提供了Queue、Pipes等多种方式来交换数据。

我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：
```python
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码:
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据进程执行的代码:
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)

if __name__=='__main__':
    # 父进程创建Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入:
    pw.start()
    # 启动子进程pr，读取:
    pr.start()
    # 等待pw结束:
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止:
    pr.terminate()
```
