---
title: 一些有意思的python代码片段
url: 129.html
id: 129
tags:
  - Python
date: 2016-08-31 17:37:54
---

本文打算长期更新一些有意思的python代码片段，就当做是学习笔记。

### 1.python多线程死锁

```python
import time,threading

locka = threading.Lock()
lockb = threading.Lock()

def fa():
    print('a')
def fb():
    print('b')

def fab():
    locka.acquire()
    try:
        fa()
        time.sleep(1)
        lockb.acquire()
        try:
            fb()
        finally:
            lockb.release()
    finally:
        locka.release()
def fba():
    lockb.acquire()
    try:
        fb()
        time.sleep(1)
        locka.acquire()
        try:
            fa()
        finally:
            locka.release()
    finally:
        lockb.release()

t1 = threading.Thread(target=fab)
t2 = threading.Thread(target=fba)
t1.start()
t2.start()
t1.join()
t2.join()
print('end')
```
### 2.协程coroutine(1)

来源：[廖雪峰](http://www.liaoxuefeng.com) 
```python
def consumer():
    r = ''
    while True:

        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

c = consumer()
produce(c)
```
运行结果为：
```
[PRODUCER] Producing 1...
[CONSUMER] Consuming 1...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 2...
[CONSUMER] Consuming 2...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 3...
[CONSUMER] Consuming 3...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 4...
[CONSUMER] Consuming 4...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 5...
[CONSUMER] Consuming 5...
[PRODUCER] Consumer return: 200 OK
```
解释：
yield表达式本身没有返回值，它的返回值需要等到下次调用generator函数时，由send(args)函数的参数赋予。
### 3.协程coroutine(2)

来源：[http://blog.csdn.net/gvfdbdf/article/details/49254037](http://blog.csdn.net/gvfdbdf/article/details/49254037) 
```
# -*- 异步IO -*-  
import asyncio  
import threading  
  
# @asyncio.coroutine把一个generator标记为coroutine类型  
@asyncio.coroutine  
def sub():  
    print('sub start: ...')  
    n = 10  
    while True:  
        print('yield start')  
        # asyncio.sleep()也是一个coroutine类型的generator，所以线程不会中断，而是直接执行下一个循环，等待yield from的返回  
        # 可以简单的理解为出现yield之后则开启一个协程(类似开启一个新线程),不管这个协程是否执行完毕，继续下一个循环  
        # 开启新协程后，print('yield start')会因为继续执行循环被立即执行，可以通过打印结果观察  
        r = yield from asyncio.sleep(1)  
        n = n - 1  
        print('---sub: %s,  thread:%s' %(n, threading.currentThread()))  
        if n == 0:  
            break  
 
@asyncio.coroutine  
def add():  
    print('add start: ...')  
    n = 10  
    while True:  
        print('yield start')  
        r = yield from asyncio.sleep(2)  
        n = n + 1  
        print('+++add: %s,  thread:%s' %(n, threading.currentThread()))  
        if n > 20:  
            break  
  
  
# 获取EventLoop:  
loop = asyncio.get_event_loop()  
# 执行coroutine  
tasks = [add(),sub()]  
loop.run_until_complete(asyncio.wait(tasks))  
loop.close()
```
执行结果：
```
add start: ...
yield start add
sub start: ...
yield start sub
---sub: 9,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start sub
+++add: 11,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start add
---sub: 8,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start sub
---sub: 7,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start sub
+++add: 12,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start add
---sub: 6,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start sub
---sub: 5,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start sub
+++add: 13,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start add
---sub: 4,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start sub
---sub: 3,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start sub
+++add: 14,  thread:<_MainThread(MainThread, started 140735116865536)>
yield start add
......
```