# python-多线程




## 讲解
###### 进程

是程序的一次执行。
每个进程都有自己的地址空间、内存、数据栈以及其他记录运行轨迹的辅助数据

###### 线程

所有的线程运行在同一个进程当中，共享相同的运行环境。
线程有：开始、顺序执行、结束 三个部分。
多个线程协同完成一个进程的任务。

> 我们在编写安全工具的时候，使用多线程要更多些(使用多进程相对较少)




### thread

> 缺点：程序复杂时，不能计算线程数量和控制，稳定性不太好

使用`_thread.start_new_thread(ping_check, (ip,))` ，第一个参数是回调函数，第二个的可变参数(tuple类型的)

Note： 必须配合`time.sleep()`

实例:

```python
"""
使用ping检查C段机器(一个C段是0-255)

"""

import _thread
import time
import re
from subprocess import Popen,PIPE


def ping_check(ip_addr):
    # 一个执行系统命令的模块
    check = Popen(['/bin/bash', '-c', 'ping -c 2 '+ip_addr], stdin=PIPE,stdout=PIPE)
    data = check.stdout.read()   # 返回的数据
    if 'ttl' in str(data):
        print("{} is up".format(ip_addr))



def main(ip_three):
    for i in range(255):
        ip = ip_three + '.' + str(i)
        _thread.start_new_thread(ping_check, (ip,))
        time.sleep(0.1)



if __name__ == '__main__':
    ip_three = input("输入ip地址的前三个字节(不需要最后一个点): \n")
    # 判断最后是否有点
    pattern = r'\.$'
    has_point = re.findall(pattern, ip_three)
    if has_point:
        ip_three = ip_three[:-1]
        print(ip_three)
    main(ip_three)
```





## threading **(重点)**

###### 1.Thead类

- 使用threading模块
- 子类化Thread类

> 解决了线程数量可控的问题

简单例子：

```python
import threading
import time
import requests


def func1(key):
    print("Hello %s:%s"%(key, time.ctime()))
    print(threading.current_thread())  # 当前线程
    print("*"*20)


def main():
    threads = []
    keys = ['张三', '李四', '王五', '陆大人']
    threads_count = len(keys)
    # 生成参数长度个线程数
    for i in range(threads_count):
        t = threading.Thread(target=func1, args=(keys[i],))
        threads.append(t)   # 线程加入线程列表

    # 启动线程
    for i in range(threads_count):
        threads[i].start()

    # 等待线程结束 join()
    for i in range(threads_count):
        threads[i].join()    # 保证【所有】的线程都会结束 再运行主线程   遇到join会阻塞



if __name__ == '__main__':
    main()
```

实例:

```python
"""
对百度以10个线程访问10次
"""

import threading
import time
import requests
import sys


def func1():
    time_start = time.time()
    r = requests.get(url='http://www.baidu.com')
    times = time.time() - time_start     # 耗时
    sys.stdout.write("Status:%s---%s---%s"%(r.status_code, times, time.strftime("%H:%M:%S")))  # 当前时间
    print("")


def main():
    threads = []
    threads_count = 10    # 定义 线程数
    # 生成参数长度个线程数
    for i in range(threads_count):
        t = threading.Thread(target=func1)
        threads.append(t)   # 线程加入线程列表

    # 启动线程
    for i in range(threads_count):
        threads[i].start()

    # 等待线程结束 join()
    for i in range(threads_count):
        threads[i].join()     # 保证【所有】的线程都会结束 再运行主线程   遇到join会阻塞




if __name__ == '__main__':
    main()
```

###### 2.生产者-消费者问题 和 Queue模块 ***(重中之重)\***

- Queue模块[ qsize(), empty(), full(), put(), get() ]
- 完美搭档：Queue + Thread

> 解决了生产参数和计算结果时间都不确定的问题

> 最常使用

**Queue：**

将产生的货物放到Queue中，消费者从Queue中拿数据

```python
import Queue

q = queue.Queue(100)  # 可以直接指定队列的大小

for i in range(10):
    q.put(i)  # 也可以这样放入

q.empty()    # 查看是否为空

q.qsize()    # 查看大小

q.get()   # 依次取出数据

q.full()   # 是否满了

queue.task_done()  # 告诉队列，这个任务执行完成了
```

生产者消费者讲解：

```python
"""
所谓，生产者与消费者模型，其实是把一个需要进程通信的问题分开考虑
生产者，只需要往队列里面丢东西（生产者不需要关心消费者）
消费者，只需要从队列里面拿东西（消费者也不需要关心生产者

生产者：
只关心队列是否已满。
没满，则生产，满了就阻塞。

消费者：
只关心队列是否为空。
不为空，则消费，为空则阻塞。

"""
from threading import Thread   # 线程
import random
import queue
import threading
import time


class Produce(Thread):
    def __init__(self, queue):
        super().__init__()
        self.queue = queue

    def run(self):
        while True:
            item = random.randint(0,99)
            if self.queue.full():
                print("当前队列长度{}".format(self.queue.qsize()))
            self.queue.put(item)   # 只要队列没满， 向队列存入数据
            print("生产者%s ==> 已经生产 %s, 并将其加入到了队列中" %(threading.current_thread(),item))


class Consumer(Thread):
    def __init__(self, queue):
        super().__init__()
        self.queue = queue

    def run(self):
        while True:
            item = self.queue.get()    # 只要队列不为空， 就从队列中取出数据
            print("消费者 ==> 从队列中取出 %s" %item)
            self.queue.task_done()  # 告诉队列，这个任务执行完成了


if __name__ == '__main__':
    q = queue.Queue(10000)
    p = Produce(q)
    p1 = Produce(q)
    c = Consumer(q)
    p.start()
    p1.start()
    time.sleep(5)
    c.start()
```

实例：

```python
"""
使用 threding 和 Quque 结合， ping_check
"""

import threading
import queue  # 注意是小写
from subprocess import Popen,PIPE
import sys


# 继承多线程的类
class DoRun(threading.Thread):
    def __init__(self, queue):
        # threading.Thread.__init__(self)
        super().__init__()
        self._queue = queue

    # 重写了run方法
    def run(self):
        # 如果Queue不为空就继续执行
        while not self._queue.empty():
            ip = self._queue.get()
            check_ping = Popen(['/bin/bash','-c','ping -c 2 '+ip], stdin=PIPE, stdout=PIPE)
            data = check_ping.stdout.read()
            if 'ttl' in str(data):
                sys.stdout.write(ip + " is up")
                print()
                
                
def main():
    threads = []
    threads_count = 10
    # 创建一个空的队列
    q = queue.Queue()

    # put到队列
    for i in range(1,255):
        q.put("123.206.96."+str(i))

    # 多线程
    for i in range(threads_count):
        threads.append(DoRun(q))

    # 启动
    for i in threads:
        i.start()

    for i in threads:
        i.join()

if __name__ == '__main__':
    main()
```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/python-%E5%A4%9A%E7%BA%BF%E7%A8%8B/  

