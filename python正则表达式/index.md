# python正则表达式



  
常用正则表达式来对爬取的网页进行解析，但是正则表达式的构造稍微复杂一点，一般在结构化的网页中没必要用正则（易出错）
  
```python
# findall
a = re.findall('jianeng',s )  ## 搜索字符串，以列表类型返回全部能匹配的子串
print(a)

#search()     # 在一个字符串中搜索，匹配正则表达式的第一个位置，返回match对象 <_sre.SRE_Match object; span=(1, 8), match='jianeng'> 只能找到第一次出现的
a = re.search('jianeng',s)
print(a)

#finditer()    # 搜索字符串，返回一个匹配结果的迭代类型，
              # 每个迭代元素是match对象

s = 'sjianengsdfasjianeng15sadfjianeng666'
finditer = re.finditer('jianeng',s)
print(finditer )

##获取match对象中的值
for i in finditer:
    print(i)
    print(i.group())     #froup() 返回匹配到的字符串
    print(i.start() )    #返回匹配的开始位置
    print(i.end() )        #返回匹配的结束位置
    print(i.span() )      # 返回一个元组表示匹配位置（开始，结束）


#sub()         # 替换 类似于字符串中 replace() 方法
s2 = re.sub('jianeng','666',s,count=2)
print(s2)


# compile()  	  # 编译正则表达式为模式对象
a = re.compile('jianeng')   # a 要匹配jianeng
dd='fsadfasfjkjianeng'
b = a.findall(dd)
print(b )

#re.split()    # 将一个字符串按照正则表达式匹配结果进行分割，返回列表类型
c = re.split('jianeng',s,maxsplit=2)
print(c)


import re

#####元字符
##通配符  -> .  点 -> 表示任意一个字符
res = re.findall(r'a','abcad')   #统一使用r取消转义  去右边字符串中找所有满足左边格式的子字符串
print(res)   #['a', 'a']

res =  re.findall(r'a..','abcad')
print(res)   #['abc']

##锚点元字符
# ^ 锁定行首
res =  re.findall(r'^a.','abcad')
print(res)    #['ab']

# $ 锁定行尾
res =  re.findall(r'a.$','abcad')
print(res)   # ['ad']

##单词边界(不是元字符)：   \b
res =  re.findall(r'dog','Just dog Monica doggie Irene')
print(res)    # ['dog', 'dog']    #第二个dog不是单独的dog单词

res =  re.findall(r'\bdog\b','Just dog Monica doggie Irene')
print(res)      # ['dog']


###重复元字符

####贪婪  （尽可能匹配的多）
res =  re.findall(r'ab{4}c','abbbbc')   #连续4个b   把花括号前面紧挨着的重复多少次
print(res)     #['abbbbc']

res =  re.findall(r'ab{2,4}c','abbbc')   #匹配2到4个b      #正则不许加空格
print(res)     #['abbbc']

res =  re.findall(r'ab{2,}c','abbbbbbbbbc')      #匹配2的到多个，至少2个
print(res)       #['abbbbbbbbbc']

###
res =  re.findall(r'ab*c','abbbc')   #   * 任意多个重复, 相当于{0,}
print(res)

res =  re.findall(r'ab+c','abbbc')   #   * 一个或多个，一到正无穷,相当于 {1,}
print(res)

res =  re.findall(r'ab?c','abbbc')   #   *    一个或者没有 ,相当于 {0,1}
print(res)

######非贪婪  (尽可能匹配的少)   后面加一个 ？
res =  re.findall(r'ab*','abbc')   # 这是贪婪
print(res)   #['abb']

res =  re.findall(r'ab*？','abbc')
print(res)   # 这是非贪婪

# *? #任意多个
# +? #一个或多个
# ?? #一个或没有


###选择元字符   |
res =  re.findall(r'ab|bc','abc-adc-aec-bca')   #二选一   #  | 或
print(res)   # ['ab', 'bc']                      #选的是正则表达式

res =  re.findall(r'a[bd]c','abc-adc-aec-bca')   # 多选一 ， [] 方括号 表示你想要列举的所有情况，但是,仅仅一个字符
print(res)   #  ['abc', 'adc']                   #选的是方括号里面的一个字符    ，  【】里面的逗号表示匹配逗号

res =  re.findall(r'a[^bd]c','abc-adc-aec-bca')   #   ^在[]里面的话，放在第一位，表示反向
print(res)   #  除了b和d

res =  re.findall(r'a[a-e]c','abc-adc-aec-bca-a^c')   #用 - 连接表示范围   [a-zA-Z0-9_]任意字母数字下划线
print(res)

####转义元字符      匹配字符本身   \ 反斜杠
res =  re.findall(r'.+.','1+2')
print(res)   #['1+2'] 不是匹配到了加号

res =  re.findall(r'1\+2','1+2')   #得到元字符的字面值
print(res)  #['1+2']

######预定义字符类
# \d  #任一数字字符   -> [0-9]
# \D  #任一非数字字符  -> [^0-9]
# \s  #任一空白符    -> [\t\n\x0B\f\r]
# \S  #任一非空白符    -> [^\t\n\x0B\f\r]
# \w  #任一字母数字字符  -> [a-zA-Z0-9]
# \W  #任一非字母数字字符  -> [^a-zA-Z0-9]


######分组
res =  re.findall(r'a(bc)+d','abcbcbcd')    #只能匹配一个bc  括号里面的变成一个整体     一个括号就是一个组
print(res)    # ['bc']

res =  re.findall(r'(a(bc)+d)','abcbcbcd')    #大组套小组
print(res)     # [('abcbcbcd', 'bc')]

res =  re.findall(r'a(b|c)c','abc-acc-adc-aec')   #与a[bc]c类似
print(res)     # ['b', 'c']

##按组提取数据
res =  re.findall(r'(\w+)-(\w+)-(\w+)-(\w+)','abc-dc-acc-aaec')
print(res)   # [('abc', 'acc', 'adc', 'aec')]

res =  re.search(r'(?P<year>\d+)-(?P<month>\d+)-(?P<day>\d+)','2018-03-13')
print(res.groupdict())    # {'year': '2018', 'month': '03', 'day': '13'}


"""
注意：
如果正则表达式中使用了括号，
那么findall函数匹配的结果
只会是括号中的内容，
而不是完整的匹配。

因此我们可以利用这种机制来
完整对需要部分的数据提取

"""
```





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/  

