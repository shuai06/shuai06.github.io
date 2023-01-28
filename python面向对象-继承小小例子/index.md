# python面向对象-继承小小例子





  
用之前学习笔记的小例子来复习一下

```python
############################继承
#父类里面有的,子类里面再写一次. 叫做重写 ,只会用子类的了,父类的就屏蔽了
#继承是通过查找来找的,  不是变量空间的复制,  还是独立的变量空间
#  多继承     既不是广度也不是深度搜索   .   mro() 方法,是在类里面的,他会自己计算出搜索顺序   例如: print(Magician.mro())或print(Magician.__mro__)  或者 print(s.__class__.mro())
#   [<class '__main__.Magician'>, <class '__main__.Role'>, <class 'object'>]
#  super  自己找父类, 如果父类改变,也不需要修改,哪怕父类被更换，也不用担心了！   super不需要再传self ,super是通过mro查找的

#__bases__特殊属性     print(Role.__bases__)     打印出来是一个元组 (<class 'object'>,)
#实例+()  自动调用__call__方法
class Role:
    def __init__(self,name,level,blood):    # 在实例化之后自动被调用,用来进行初始化
        self.name = name
        self.level = level
        self.blood = blood

    def __repr__(self):
        return "{cls}:{name},{level},{blood}".format(cls=self.__class__.__name__,
                                                          name=self.name,
                                                          level=self.level,
                                                          blood=self.blood )

    def figth(self):
        raise NotImplementedError('必须在子类中实现该行为')


class SwordsMan(Role):    #继承了Role类
    def __init__(self,name,level,blood,attack_power):    #重写了__init__
         #Role.__init__(self,name,level,blood)
         super().__init__(name, level, blood)   #不在明确指定类名
         self.attack_power = attack_power

    def fight(self):
        print('物理攻击')

class Magician(Role):     #继承自Role
    def __init__(self,name,level,blood,magical_power):              #重写了__init__
        #Role.__init__(self,name,level,blood)
        super().__init__(name, level, blood)  # 不在明确指定类名
        self.magical_power = magical_power

    def fight(self):
        print('魔法攻击')

    def cure(self):
        print('治疗')


## 鸭子类型
#展示出攻击的效果(这件事,不属于任何一个角色)
#写出一个函数
def draw_fight(role):    # 鸭子类型体现出来是,一个函数,所关心的不是类型,而是行为
    print('role', end='')
    role.fight()  # 其实并不关心,role是不是一个role实例

class GirlFriend:    #虽然没有继承Role类,依然可以用鸭子类型
    def __init__(self,name):
        self.name = name

    def fight(self):
        print('拔电源攻击')

g = GirlFriend('xxx')
draw_fight(g)    #打印出role拔电源攻击

s= SwordsMan('xps',18,3000,150)   #实例化
m = Magician('ps',18,1400,6660)
draw_fight(s)   #打印出 role物理攻击
draw_fight(m)   #打印出 role魔法攻击

print(s)
print(m.__repr__())





print('***********************************************************')
# 继承是通过查找来找的,  不是变量空间的复制,  而是独立的变量空间
class BaseClass:
    attribute = 1

    def method(self):
        pass

class DerivedClass:    # 简单继承,没有任何的封装与重写
    pass

print(DerivedClass.__dict__)   # DerivedClass中没有 attribute 与 method
print(BaseClass.__dict__)     # BaseClass中有
print(Magician.__mro__)
print(s.__class__.mro())

```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/python%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1-%E7%BB%A7%E6%89%BF%E5%B0%8F%E5%B0%8F%E4%BE%8B%E5%AD%90/  

