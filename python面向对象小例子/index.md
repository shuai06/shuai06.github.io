# python面向对象小例子



  
用一个银行存款取款的例子来回忆一下面向对象最基础的部分

```python

class Account:
    all_count={}   #类属性
    def __init__(self, name, number, balance):  # 初始化账户的'实例方法',见名知意
        self.name = name
        self.number = number  # 实例的变量属性
        self.balance = balance
        self.all_count[self.number] = self   #

    # def __del__(self):  #当实例对象被销毁的时候,当变量的指向为0的时候  自动触发
    #     print(self.name,'被销毁了')


    def deposit(self, amount):  # 存款
        if amount <= 0:
            print('存款金额不得为负')
        else:
            self.balance += amount

    def withdraw(self, amount):  # 取款
        if amount > self.balance:
            print('余额不足')
        else:
            self.balance -= amount

    def describe(self):  # 查询账户详情
        return "Account(姓名:{name},账户:{number},余额:{balance})".format(name=self.name,
                                                           number=self.number,
                                                           balance=self.balance)
    def close(self):
        self.all_count.pop(self.number)


a=Account('xps','6217',10000)
v = getattr(a,'nme','000')
# b = a
# del a
print(v)
# a.deposit(5000)
# a.withdraw(2500)
# print(a.describe())
#
# print(a.__dict__)
# print(__name__)

############################################ 笔记 #######################################3
#__init__()    在实例化之后自动被调用,用来进行初始化
#__del__()      删除变量的指向,不是直接去删除这个对象  例如:a=[1,2,3]  b=a   del a   a变量没了,但是b还存在
                #程序执行完之后变量的内存自动释放.自动触发  __del__

#__str__()     #str 会自动触发    对用户友好   后面可以直接 print(a) 来输出str的值,如果没有定义a,则输出repr的值
#__repr__()    repr会自动触发     对开发者友好     在控制台shell 直接 a  输出repr的值  ,如果repr没有定义,则不会找str

#getattr()      getattr(i,'name','没有') 相当于转换成对象的访问 obj.name  获取,,和字典里面的get()方法类似,找不到返回'没有'   'name'这里是一个字符串
#hasattr()      hasattr(i,'xxx')  返回布尔值 (还可以避免因为属性没有而产生报错)   'xxx'也是字符串       hasattr 还可以避免，因为属性没有，而导致的报错


#setattr()      setattr(i,'name','vvv'')  自动转换成i.name = 'vvv'         'name'这里是一个字符串
#delattr()      delattr(i,'name')     删除                                   'name'这里是一个字符串    转化成del i.name

#__class__      可以得到一个类实例所属的类对象,print(a.__class__)   <class '__main__.Account'>   不是类名
#__doc__       注释文档写了之后,可以help出来   如:print(help(Account))
                #  注释存储在了__doc__里   如: print(Account.__doc__)可以打印其文档注释

#__dict__      一个存储了对象属性的字典 把该变量空间的打印出来
                #  例:print(a.__dict__)  >>{'balance': 12500, 'name': 'xps', 'number': '6217'}
                #类也可以   print(Account.__dict__)
#__name__

def dance():
    pass
print(dance.__name__)     #可以查看该函数的名字

class C:
    pass
print(C.__name__)         #可以查看该类的名字

#类名  是一个字符串

#print 输出的是你给他的东西得  __str__

#shell 里面  输出的是  你给他的东西的 __repr__

#self   是实例本身
# print 输出的str 但是如果输出一个数据结构(集合元组字典列表)的数据,里面的数据,会以repr展示


########################################################################################
if __name__ == '__main__':
    pass



#print(str(__name__))       __name__

#import 的时候,会把那个文件执行一遍
###如果作为一个模块被导入的话,就不会调用__name__以下的东西
#  print(__name__)     结果为  __main__

#######################################################闭包###########################33
class A:
    def __del__(self):
        print('被销毁0')

def outer():   #
    a=A()  #a 是在函数调用时候实例化的
    def inner():    #inner 也是在outer里面定义的,在outer结束的时候inner也被销毁
        print(a)
    return  inner

x = outer()     #用一个变量去接受,就不会打印出来,inner获得一个临时指向
del x #销毁函数inner




#1.构造局部的全局变量
#2.在没有类的情况下,封装变量


input()



######私有化,保护作用,类的外面访问不到
_eye=2  # protect  隐藏  但是可print(dog._eye)访问.修改

__leg=4    # 彻底保护.  万一访问的话,可以以下在外面访问(先定义一个方法):
def getLeg(self):     #获取私有属性
    return self.__leg
leg = dog.getLeg()
print(leg)     #可通过此方法在类外面查看

###如果是修改私有属性的话:
def setLeg(self,leg):    #设置
    self.__leg = leg
dog.setLeg(55)    #修改私有属性
print(dog.getLeg())   #用get 方法来查看一下


```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/python%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E5%B0%8F%E4%BE%8B%E5%AD%90/  

