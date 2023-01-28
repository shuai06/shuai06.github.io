# python-自带的线程池和进程池


  
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
p.join()


# 进程池比线程池耗费资源



# 可以将返回值.get() 但是get也会i阻塞
async_result = p.apply_async(func) # 函数
print(async_result.get())
```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/python-%E8%87%AA%E5%B8%A6%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%92%8C%E8%BF%9B%E7%A8%8B%E6%B1%A0/  

