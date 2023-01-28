# python-进程



  
**并发：**   任务数大于cpu个数

**并行：**    cpu个数和任务数相同



**GIL 锁:**   任何python进程中，一次永远只有一个线程运行

一个python进程  只能执行一个线程



###### 创建进程

```python
## 不同操作系统创建进程的区别：

#linux上:   fork() 
import os
import time

pid = os.fork() # 这个地方会创建一个子进程， 他的pid号值永远为0
if pid == 0:
    print("我是子进程")
    time.sleep(3)
else:
    print("我是父进程")
    time.sleep(3)




# win上创建进程 类似于导入机制   都通用的
import multiprocessing

def func():
    print("000")

if __name__ == '__main__':
    p = multiprocessing.Process(target=func)  # 生成进程, 传参target= ，args=
    p.start()       # 开启进程， 相当于一个子进程



```







###### 进程标识

```python
# 进程标识  pid

import time
import os
import multiprocessing

def func():
    time.sleep(5)

if __name__ == "__main__":
    print("main pid ", os.getpid())   # 当前进程的
	print(multiprocessing.current_process().pid)
    p = multiprocessing.Process(target=func)
    print(p.pid)
    p.start()     # start之后才有pid号
    print(p.pid)
    time.sleep(10)
# 线程的标识是 ident


# 操作系统调用的是进程
"""
注意！
操作系统并不能看到线程的标识。
因为，线程是由Python解释器
来负责调度的。

操作系统仅需要调度进程就行了
"""
```



###### 守护进程

```python
# 守护进程  daemon = True  主进程结束之后，子进程跟着结束（某子进程的生命周期随着主进程）
import time
import multiprocessing

def func():
    time.sleep(20)
    print("子进程结束")

if __name__ == "__main__":
    p = multiprocessing.Process(target=func, daemon = True)
    p.daemon = True   # 设置成守护进程 p True会随着主进程结束而结束， 主进程不会等待子进程结束
    p.start()
    time.sleep(3)

```



###### 终止进程

```python
# 终止进程
import multiprocessing
import time

def func():
    time.sleep(5)
    print(multiprocessing.current_process())


if __name__ == "__main__":
    p = multiprocessing.Process(target=func)
    p.start()
    time.sleep(2) # 主进程2s
    p.terminate()   # 结束,主进程结束，不管子进程有没有结束，就终止子进程（线程没有这个）


```



###### 面向对象

```python
# 面向对象 类继承创建进程
# start() --> run(已经是在新的进程了) --> target   # target是由默认的run运行
import multiprocessing
import time

class My_Process(multiprocessing.Process):

    def __init__(self, *args, **kwargs):
        super().__init__(args, kwargs)
        print("初始化...")

    def run(self):
        """
        start 默认调用的方法   重写啦在这里
        :return:
        """
        print("run...")
        self.task()
        time.sleep(5)

    def task(self):
        print("task...")


if __name__ == "__main__":
    p = My_Process()
    p.start()   # start 方法会调用run

# 创建进程的方式 ： multiprocessing.Process     类继承，重写run     linux下：fork
# 如果换成是线程的话：换掉继承类就行

```



###### 进程通信

```python
# -*- coding:utf-8 -*-

import multiprocessing
from multiprocessing import Manager  # 管理器


def func(l):
    l.append(111)

if __name__ == '__main__':
    manager = Manager()  # 实例化  先开启一个公共进程，并返回一个管理器
    l = manager.list()   # 开启空间，左边就是代理
    # l = manager.dict()
    """
    一般常用的空间类型是：
    1.  mgr.list()
    2.  mgr.dict()
    3.  mgr.Queue()
    """
    print(l)
    p = multiprocessing.Process(target=func, args=(l,))
    p.start()
    p.join()
    print(l)   # 这样使用manager之后 l就是【共享】的了
    
```



进程池 & 线程池

ps:有点乱

```python
# from multiprocessing import Pool   # 进程池
from multiprocessing.dummy import Pool  #线程池
from multiprocessing.pool import ThreadPool # 线程池
import threading
import time


def func(i):
    print("{}-------555".format(i))
    time.sleep(2)
    return i

def print_back(*args, **kwargs):
    print("处理数据完成",args, kwargs)

pool = Pool(6)   # 不写的话 默认是cpu的个数
# print(threading.active_count())

for i in range(5):
    pool.apply_async(func=func, args=(i,), callback=print_back)  ## 添加任务   不阻塞    主要使用的方法
#     pool.apply(func=func, )  ## 添加任务   阻塞
# pool.map(func, [i for i in range(5)])   #添加任务  不阻塞

pool.close()   #关闭线程池  不在提交新的任务
pool.join()    #等待进程池中的任务执行完毕
print("任务结束")


########### 线程池的步骤
p = ThreadPool(3) # 实例化
p.apply_async(func) # 函数      # 可以将返回值.get() 但是get也会i阻塞
p.close()
p.join()   # join()语句要放在close()语句后面


# 进程池比线程池耗费资源


# 可以将返回值.get() 但是get也会i阻塞
async_result = p.apply_async(func) # 函数
print(async_result.get())

```



###### 使用进程池来实现并发服务器

```python
# 使用池来实现并发服务器
# 先开一个进程池， 每个进程下面再开一个线程

import socket
from multiprocessing import Pool, cpu_count
from multiprocessing.pool import ThreadPool


def woeker_thread(conn):  # 使用线程池来
    while True:
        recv_data =conn.recv(1000)
        if recv_data:
            print(recv_data)
            conn.send(recv_data)
        else:
            conn.close()
            break

def worker_process(server): # 使用进程池来接收套接字
    # pool = Pool(cpu_count()*2)     # 通常可以分配2倍的cpu个数
    pool = ThreadPool(cpu_count()) # 获取电脑核心数
    while True:
        conn, addr = server.accept()
        pool.apply_async(woeker_thread, args=(conn,))


if __name__ == '__main__':
    server = socket.socket()
    server.bind(('127.0.0.1', 8888))
    server.listen(1000)
    n = cpu_count()  # 获得当前计算机的cpu核心数量
    pool = Pool(n)
    for i in range(n):  # 充分利用cpu，为每个cpu分配一个进程
        # conn, addr = server.accept()
        pool.apply_async(func=worker_process, args=(server,))
        pool.apply_async(func=worker_process, args=(server,))
    pool.close()
    pool.join()

```







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python-%E8%BF%9B%E7%A8%8B/  

