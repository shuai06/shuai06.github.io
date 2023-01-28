# python基础-装饰器


  


  
###### 装饰器

```python
###函数的装饰器,本质上就是一个闭包
## 概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。
def  f1(func):
    print('f1 runing')
    def f2(y):
        print('f2 running')
        return  func(y) +1
    return f2

@f1    #装饰器 (@符号加函数名,装饰上了gun函数)   #相当于f1(gun)
def gun(m):
    print('gun runing')
    return m*m

## @f1  ->   f1( gun) -> f2
n=gun(5)    #   gun(5) -> 就是在运行  f2(5)
print(n)


#############例子:
import time
def run_time(func):
    def new_fun(*args,**kwargs):
        t0 = time.time()
        print('star time: %s'%(time.strftime('%X',time.localtime())) )
        back = func(*args,**kwargs)
        print('end time: %s'%(time.strftime('%X',time.localtime())) )
        print('run time: %s'%(time.time() - t0))
        return back
    return new_fun


@run_time
def test2(n):
    for i in range(1,n):
        for j in range(1,i+1):
            print('%dx%d=%2s'%(j,i,i*j),end = ' ')
        print ()
#@run_time -> run_time(test2)  -> new_fun
n = test2(10)  #就相当于运行new_time(5)
```

###### 例子

```python
# 写一个login函数  然后写一个装饰器，模拟登录过程：让程序延迟3秒 在延迟过程中输出正在验证
"""
def login():
    print('登录成功')
"""
import time

def login_required(func):
    def wrpper(*args,**kwargs):
        print('正在验证...')
        time.sleep(5)
        return func(*args,**kwargs)
    return wrpper

@login_required
def login():
    print('登录成功')
   #return  'code:200'

login()
```



##### 类作为装饰器

```python
## 用__call__方法

class Tset_Class():
    def __init__(self,func):  # 传进来函数
        print('正在实例化')
        self.func = func
    def __call__(self, *args, **kwargs):   #类装饰器, 一定要写__call__方法
        print('这是call方法')
        return self.func

@Tset_Class
def  test():
    print('这是一个测试函数')

n =  test()   #    self.func : test 函数体
n()           #   n()  调用test（）

#  1.  @Tset_Class  ：  Tset_Class( test )  -> 相当于：t = Tset_Class( test )
#  2.  test()  ->  相当于：   调用实例的call方法 -> 返回 self.func函数体
#  3.  n =  test()   ，n()  调用test（）
```



###### 类中常见的几种装饰器

```python
class Test():
    aa = 123

    def __init__(self,name):
        self.__name = name

    @property  # 把方法变成属性
    def get_name(self):
        print( self.__name)
        return  self.__name

    @get_name.setter   # 可以让get方法，变成set方法
    def get_name(self,name):
        self.__name = name

    @property  # 把方法变成属性
    def po(self):
        print('asdfasdfasdfasdaf')

    @staticmethod
    def test():
        print('staticmethod  func')

    @classmethod
    def show(cls):
        print(cls.aa)  # 可以访问类属性
        print('classmethod func ')

t = Test('jianeng')

t.show()
# @property
# t.get_name = 4   # get_name.setter 没有写的时候， = 4 会报错
# t.get_name
# t.po


# @staticmethod     #没有self
t.test()   # 没有self  ，实例不能调用。
Test.test()  # 类名可以直接调用


###  @classmethod    #类方法, 一般是用来做封装  class名字已经传进去了
Test.show()  #有self， 类名不能直接调用
t.show()
##静态方法装饰器： staticmethod    类调用与实例调用无差别， 也不需要self参数，类似于普通函数   不需实例化就可以调用,直接用类名调用   （与实例解绑）， 不能访问其他的类属性与方法（相当于类的方法）

##类方法装饰器： classmethod        self参数被换成cls参数， 自动传入对应的类  可以访问类属性与其他类方法
```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/python%E5%9F%BA%E7%A1%80-%E8%A3%85%E9%A5%B0%E5%99%A8/  

