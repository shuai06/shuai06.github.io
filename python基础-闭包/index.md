# python基础-闭包






```python
# 闭包:   一个函数里面嵌套一个函数,调用外层函数返回里层函数本身
#1.
def fx(x):
    x+=1
    def fy(y):
        return  x*y
    return  fy     #不要加括号

f=fx(5)      #拿到fy的函数体
n = f(5)      #fy()
print(n)


#2.传进去的参数为函数体
def  f1(func):
    print('f1 runing')
    def f2(y):
        print('f2 running')
        return  func(y) +1
    return f2

def gun(m):
    print('gun runing')
    return m*m

temp =  f1(gun)   # 返回f2函数体   # f1 runing  -> f2  ->temp
n= temp(5)     # f2( 5)  ->  ' f2 running '  -> return  func(y) +1
            #  func(y) ===  gun(m) -> gun runing   ->  5*5
            #  return  25 +1  -> 26

print(n)

```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python%E5%9F%BA%E7%A1%80-%E9%97%AD%E5%8C%85/  

