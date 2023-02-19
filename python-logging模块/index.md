# python-logging模块






###### 为什么会用logging模块

- 灵活性好，方便配置
- 输出或保存不同级别日志

###### logging模块结构

logging 在源码中有三个文件,结构如下:

```
├── config.py
├── handlers.py
└── __init__.py
```

- `__int__.py`中实现了基础功能,主要的逻辑就在这个文件中
- `handlers.py`是一些Handlers用起来很方便的.
- `config.py `是对配置做处理的方法.




###### 基础

```python
import logging


# 1.初始化   相当于
logger = logging.getLogger("test_name")


# 2.设置级别
logger.setLevel(logging.DEBUG)  # 低于这个级别就不去管他


# 3. 定义handler:
# 在控制台输出  FileHandler
sh = logging.StreamHandler()   # 定义控制台输出 
sh.setLevel(logging.ERROR)   # 设置最低级别 达到什么级别的时候管 低于..不执行  

# 写在文件里面 SteamHandler
fh = logging.FileHandler(r'file_test.log')
fh.setLevel(logging.DEBUG)  # 达到什么级别就写到文件里面去


# 4. 格式化输出   formatter     注意之间逗号不需要加，只是为了美观
formatter = logging.Formatter(
   '时间：%(asctime)s,'
   '日志级别:%(levelname)s,'
   '日志信息：%(message)s,'
   )
sh.setFormatter(formatter)  # 设置控制台的样式
fh.setFormatter(formatter)  # 设置文件的样式


# 添加进去
logger.addHandler(sh)
logger.addHandler(fh)




if __name__ == '__main__':
   logger.debug('测试中')
   logger.info('正常运行')
   logger.warn('警告')
   logger.error('完了error')
   logger.critical('炸了')

   # 例如：
   def func(a):
      try:
         num = 40/a
         logger.info(num)   # 正常的话记录
      except Exception as e:
         logger.error(e)   # 将错误记录
      
   func(0)




##  logging 中的级别
"""
DEBUG：调试信息，通常在诊断问题的时候用得着；
INFO：普通信息，确认程序安装预期运行；
WARNING：警告信息，表示发生了意想不到的事情，或者指示接下来可能会出现一些问题，但是程序还是继续运行；
ERROR：错误信息，程序运行中出现了一些问题，一些功能没有执行；
CRITICAL：危险信息，一个严重的错误，导致程序无法继续运行。

# 对应下面5个方法
debug
info
Warning
error  
critical 
"""


```



###### logging常用的格式化:

<img src="https://image.geoer.cn/logging%E5%B8%B8%E7%94%A8%E7%9A%84%E6%A0%BC%E5%BC%8F%E5%8C%96.jpg"></img>













---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python-logging%E6%A8%A1%E5%9D%97/  

