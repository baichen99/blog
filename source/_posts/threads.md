---
title: 线程
date: 2018-06-17 16:31:04
categories:
- python
tags:
- python
- 线程
preview: 100
---


Python标准库提供了两个模块: thread 和 threading, thread是低级模块, threading是对其的封装, 绝大多数情况下我们只需要使用threading
<!-- more -->
## 使用threading创建多线程
有两种方法

第一种方法是:把一个函数传入并创建Thread实例,然后调用start方法开始执行

第二种方法是:从threading.Thread**继承**并创建线程类, 然后**重写**__init__方法和run方法

方法一演示如下
```python
import random
import time, threading

def thread_run(urls):
    print("Current process %s is running" % threading.current_thread().name)
    for url in urls:
        print("%s ------> %s" % (threading.current_thread().name, url))
        time.sleep(1)
    print("%s ended" % threading.current_thread().name)


print("%s is running..." % threading.current_thread().name)
t1 = threading.Thread(target=thread_run, name="thread1", args=(['1', '2', '3', '4'],))
t2 = threading.Thread(target=thread_run, name="thread2", args=(['5', '6', '7', '8'],))

t1.start()
t2.start()
t1.join()
t2.join()

print("%s ended" % threading.current_thread().name)
```
运行结果
```
MainThread is running...
Current process thread1 is running
thread1 ------> 1
Current process thread2 is running
thread2 ------> 5
thread1 ------> 2
thread2 ------> 6
thread1 ------> 3
thread2 ------> 7
thread2 ------> 8
thread1 ------> 4
thread1 ended
thread2 ended
MainThread ended

Process finished with exit code 0

```
方法二
```python
import random, threading, time

class MyThread(threading.Thread):


    def __init__(self, name, urls):
        threading.Thread.__init__(self, name=name)
        self.urls = urls

    def run(self):
        print("Current %s is running..." % threading.current_thread().name)
        for url in self.urls:
            print("%s --->>> %s" % (threading.current_thread().name, url))
            time.sleep(1)
        print("%s ended" % threading.current_thread().name)
print("%s is running" % threading.current_thread().name)
t1 = MyThread(name="Thread1", urls=['1', '2', '3'])
t2 = MyThread(name="Thread2", urls=['4', '5', '6'])
t1.start()
t2.start()
t1.join()
t2.join()
print("%s ended" % threading.current_thread().name)
```
运行结果如下
```
MainThread is running
Current Thread1 is running...
Thread1 --->>> 1
Current Thread2 is running...
Thread2 --->>> 4
Thread2 --->>> 5
Thread1 --->>> 2
Thread2 --->>> 6
Thread1 --->>> 3
Thread2 ended
Thread1 ended
MainThread ended

Process finished with exit code 0

```
## 线程同步
多个线程对某个数据进行修改可能会出现错误, 使用Thread对象的Lock和RLock方法可以实现简单的线程同步,这两个对象都有acquire方法和release方法, 对于哪些每次只允许一个线程操作的数据, 可以将其操作放到acquire和release之间
对于**Lock对象**如果一个线程进行两次aquire,由于没有release第二次的aquire将会挂起,造成死锁
**RLock**对象允许一个线程多次对其进行acquire操作
```python
# RLock使多个线程不能同时更改变量
import threading
mylock = threading.RLock()
num = 0


class MyThread(threading.Thread):
    def __init__(self, name):
        '''
        初始化
        :param name: 线程名称
        '''
        threading.Thread.__init__(self, name=name)


    def run(self):
        global num
        while True:
            mylock.acquire()
            print("%s locked, Number: %d" % (threading.current_thread().name, num))
            if num >= 4:
                mylock.release()
                print("%s released, Number: %d" % (threading.current_thread().name, num))
                break
            num += 1
            print("%s released, Number: %d" % (threading.current_thread().name, num))
            mylock.release()


if __name__ == '__main__':
    thread1 = MyThread('thread1')
    thread2 = MyThread('thread2')
    thread1.start()
    thread2.start()
```
执行结果
```
thread1 locked, Number: 0
thread1 released, Number: 1
thread1 locked, Number: 1
thread1 released, Number: 2
thread1 locked, Number: 2
thread1 released, Number: 3
thread1 locked, Number: 3
thread1 released, Number: 4
thread1 locked, Number: 4
thread1 released, Number: 4
thread2 locked, Number: 4
thread2 released, Number: 4

Process finished with exit code 0

```
