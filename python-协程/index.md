# python-协程




#### 概念

###### 1.进程(同步)

进程是系统进行资源分配和调度的一个独立单位。每个进程都有自己的独立内存空间，不同进程通过进程间通信来通信。

上下文进程间的切换开销，比较大，但相对比较稳定安全

###### 2.线程（同步）

线程是CPU调度和分派的基本单位。

线程间通信主要通过共享内存，上下文切换很快，资源开销较少，但相比进程不够稳定容易丢失数据。

###### 3.协程（异步）

**协程是一种用户态的轻量级线程，**协程的调度完全由用户控制

“协程就是你可以暂停执行的函数”。协程拥有自己的寄存器上下文和栈。

`非抢占式多任务`



###### 协程与线程的区别:

1) 一个线程可以多个协程，一个进程也可以单独拥有多个协程。

2) 线程进程都是同步机制，而协程则是**异步**。

3) 协程能保留上一次调用时的状态，每次过程重入时，就相当于进入上一次调用的状态。

4）线程是抢占式，而协程是**非抢占式**的，所以需要用户自己释放使用权来切换到其他协程，因此同一时间其实只有一个协程拥有运行权，相当于单线程的能力。

5）协程并不是取代线程, 而且抽象于线程之上, 线程是被分割的CPU资源, 协程是组织好的代码流程, 协程需要线程来承载运行, 线程是协程的资源, 但协程不会直接使用线程, 协程直接利用的是执行器(Interceptor), 执行器可以关联任意线程或线程池, 可以使当前线程, UI线程, 或新建新程.。

6）线程是协程的资源。协程通过Interceptor来间接使用线程这个资源。

#### 代码

**greenlet**

```python
# 协程   =>又叫：非抢占式多任务
# 中断执行
# 线程
"""
pip install greenlet
"""

# 基本使用
from greenlet import greenlet
import random
import time


def Producer():
    while True:
        item = random.randint(0, 99)
        print("pro生产了{}".format(item))
        c.switch(item)   # 暂停当前协程,切换到(执行的协程)消费者，并将item传入消费者
        time.sleep(1)

def Consumer():
    while True:
        item = p.switch()    # 切换到生产者， 并等待消费者传入item
        print("消费了{}".format(item))

c = greenlet(Consumer)   # 将一个普通函数变成协程
p = greenlet(Producer)
c.switch()      # 让消费者先进入暂停状态， （只有恢复的时候才能接受数据）




"""
# greenlet 的价值
价值一： 高性能的原生协程
价值二： 语义更加明确的显式切换
价值三： 直接将函数包装成协程，保持原有代码风格
"""

```



**gevent**

gevent是一个基于协程的python网络库，在遇到IO阻塞时，程序会自动进行切换，可以让我们用同步的方式写异步IO代码。

```python
# gevent   
# 当一个greenlet遇到IO操作，比如访问网络，就会自动切换，直到IO操作完成，再切换回来
# 遇到阻塞就切换到另一个协程继续执行 ！
"""
pip install gevent

gevent，通过封装了 libev（基于epoll） 和 greenlet 两个库。
帮我们做好封装，允许我们以类似于线程的方式使用协程。
以至于我们几乎不用重写原来的代码就能充分利用 epoll 和 协程 威力
"""

import gevent
from gevent.queue import Queue

#  猴子补丁   monkey    将一些阻塞的模块动态的修改为非阻塞
from gevent import monkey;monkey.patch_all()  # 不会阻塞了就
from gevent import monkey;monkey.patch_socket()  # 不会阻塞了就
import requests
import gevent

def get_response():
    print("start")
    requests.get("https://www.baidu.com")
    print("end")

tasks = [gevent.spawn(get_response) for i in range(20)] # 创建协程
gevent.joinall(tasks)    # 阻塞等待协程完毕

# ---------------------------------------------------------------------

### gevent 并发服务器
import gevent
# 将python内置的socket直接换成了IO多路复用的socket
from gevent import monkey;monkey.patch_socket()
import socket

server = socket.socket()
server.bind(('127.0.0.1', 8888))
server.listen(1000)

def worker_coroutine(conn):
    while True:
        recv_data = conn.recv(1000)
        if recv_data:
            print(recv_data)
            conn.send(recv_data)
        else:
            conn.close()
            break

if __name__ == '__main__':
    while True:
        conn, remote_addr = server.accept()
        # 生成一个协程， 并将conn作为参数传入
        gevent.spawn(worker_coroutine, conn)  


# ---------------------------------------------------------------------

# gevent 协程通信
"""
问题一： 协程之间不是能通过switch通信嘛？
是的，由于 gevent 基于 greenlet，所以可以。

问题二： 那为什么还要考虑通信问题？
因为 gevent 不需要我们使用手动切换，
而是遇到阻塞就切换，因此我们不会去使用switch ！
"""

import gevent
from gevent import monkey;monkey.patch_all()
from gevent.queue import Queue
import random

queue = Queue(3)

def Producer():
    while True:
        item = random.randint(0, 99)
        print("生产了{}".format(item))
        queue.put(item)

def Consumer():
    while True:
        item = queue.get()
        print("消费了{}".format(item))


p = gevent.spawn(Producer, queue)  # 将函数封装成协程， 并开始调度
c = gevent.spawn(Consumer, queue)

# gevent.sleep(5)
gevent.joinall([p, c])    # 阻塞（一阻塞就切换协程） 等待   等待你带进来的所有协程对象结束
```



> Note:

```python
# 这三句一定要放在最上面
import gevent
from gevent import monkey
monkey.patch_all()  
```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python-%E5%8D%8F%E7%A8%8B/  

