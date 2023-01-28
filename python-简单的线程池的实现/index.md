# python-简单的线程池的实现



  
```python
# 多进程服务器   创建 销毁
# 通过提前创建好线程  当任务来了 就分配线程去执行

"""
主线程： 相当于生产者，只管向线程池提交任务。
               并不关心线程池是如何执行任务的。
               因此，并不关心是哪一个线程执行的这个任务。
线程池： 相当于消费者，负责接收任务，
               并将任务分配到一个空闲的线程中去执行。
"""

from threading import Thread
from queue import Queue
import time


class ThreadPool:
    def __init__(self, n):  # 意味着传啦几次
        self.queue = Queue()
        for i in range(n+1):   # 每个线程都去执行类里面的func方法
            Thread(target=self.work, args=(self.queue, ), daemon=True).start()

    def work(self):
        while True:
            func, args, kwargs = self.queue.get()
            func(*args, **kwargs)
            self.queue.task_done()

    def apply_async(self, func, args=(), kwargs={}):   # 主线程调用的
        self.queue.put((func, args, kwargs))   # 扔到队列里面

    def join(self):
        self.queue.join()

def task1():
    time.sleep(2)
    print("111")

def task2():
    time.sleep(2)
    print("222222")

pool = ThreadPool(2)
pool.apply_async(task1)
pool.apply_async(task2)
print("任务提交完成")
pool.join()
print("任务完成")
```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/python-%E7%AE%80%E5%8D%95%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E5%AE%9E%E7%8E%B0/  

