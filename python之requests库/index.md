# python之requests库


Requests是基于urllib3来改写的，采用Apache2 Licensed 来源协议的HTTP库。

它`比urllib更加方便`，可以节约我们大量的工作，完全满足HTTP测试需求。


  
## 安装

`pip install requests`


  
## 使用
**测试网站**：`http://httpbin.org/`

###### GET请求
```python
import requests

# 不带参数
r = requests.get('http://httpbin.org/get')
print(r.text)

# 携带参数
data = {
    'name':'zhangsan',
    'age':20,
}
r1 = requests.get('http://httpbin.org/get', params=data)
r3.encoding = 'utf-8'   # 设置编码
print(r1.text)


#如果是中文或者有其他特殊符号，则进行url编码
from urllib.parse import urlencode
job = "程序员"
parm = urlencode(job, encoding="utf-8")
# 再进行拼接后进行后续请求...
```



###### POST请求

```python
import requests

# 不带参数
r = requests.post('http://httpbin.org/post')
print(r.text)


# 携带参数
data = {
    'user': 1,
    'pwd': 'hhh',
    'submit': 'submit',
}
r1 = requests.post('http://httpbin.org/post', data=data)
```



###### 请求头

```python

# 自定义请求头
headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.106 Safari/537.36',
}
response1 = requests.get(url, headers=headers,params=data)


# 打印请求头
print(response1.request.headers)

# 两种遍历方式
for key, value in r3.headers.items():
    print(key+":"+value)
print("************************************")
for key in r3.headers:   # 不是请求头
    print(key + ":" + (r3.headers)[key])
```





###### 响应信息

```python
# 响应头
print(response1.headers)

# 其他信息
print(response1.headers['Server'])
print(response1.url)   # 打印url
print(response1.status_code)   # 状态码 200
print(response1.cookies.items())   # cookie
print(response1.apparent_encoding)  # 提供的编码
print(response1.elapsed)  # 请求到响应的时间
print(r.history)    # history属性得到请求历史

# cookie操作
print(response1.headers['Set-Cookie'])

print(response.text)    #   响应返回的数据: str
print(response.content)   #  响应返回的数据: 二进制形式

# 对文件内容查找(比如被安全狗拦截后页面中会有关键字:`safedog`)
...
if response.text.find('safedor'):
    print('被拦截')
else:
    print('绕过')
...
    


# 比如爬图片，音乐，视频就是用的二进制数据
import requests
response = requests.get('http://xpshuai.cn/favicon.ico')
# 保存图片  wb写入
with open('favicon.ico','wb')as f:
    f.write(response.content)
```



###### 解析json

使用json（）方法，就可以将返回结果是JSON格式的字符串转化为字典

```python
import requests

r = requests.get('http://httpbin.org/get')
print(r.text)
print(type(r.text))   # <class 'str'>
print(r.json())
print(type(r.json()))  # <class 'dict'>
```



###### SSL证书检验

```python
# 参数  verify=False时表示，对于 https 不验证证书
 
r = requests.get(url1, verify=False)
    
```



###### 设置代理

```python
import requests

proxies = {'http': ' http://127.0.0.1:8080', 'https': ' https://127.0.0.1:8080'}

r = requests.get(url, proxies=proxies, verify=False)
```

可以设置代理池, 随机更换

```python
proxies = []

request = requests.get(url, proxies={'http': random.choice(proxies)}, headers=head) 

```







###### 自定义cookie

```python
#打印cookie
for key,value in r.cookies.items():
    print(key + '=' + value)
```



1.简单的做法:  

```python

import requests
 
headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/"
                 "537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36"
}
 
cookies = {"cookie_name": "cookie_value", }
response = requests.get(url, headers=headers, cookies=cookies)


```

2.CookieJar

更专业的方式:先实例化一个RequestCookieJar的类，然后把值set进去，最后在get,post方法里面指定cookies参数

```python


from http.cookiejar import CookieJar


requests.cookies.update()

c = requests.cookies.RequestsCookieJar()
c.set('cookie-name', 'cookie-value', path='/', domain='.abc.com')
s.cookies.update(c)
```

3.requests.Session()方法

```python
session = requests.Session()
session.cookies['cookie'] = 'cookie-value'

# 功能：可以添加cookie，不会清除原cookie
# 缺点：不能设置path，domain等参数
```





###### session

**会话维持**

在requests中，如果直接利用get（）或post（）等方法的确可以做到模拟网页的请求，但是这实际上是相当于不同的会话，也就是说相当于你用了两个浏览器打开了不同的页面。

```python
import requests
 
 
headers = {
    "content-type": "application/x-www-form-urlencoded;charset=UTF-8",
    "User-Agent" : "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9.1.6) ",
}
 
#设置一个会话session对象s
s = requests.session()
resp = s.get('https://www.baidu.com/s?wd=python', headers=headers)
# 打印请求头和cookies
print(resp.request.headers)
print(resp.cookies)
 
# 利用s再访问一次
resp = s.get('https://www.baidu.com/s?wd=python', headers=headers)
 
# 请求头已`保持`首次请求后产生的cookie
print(resp.request.headers)

```





###### 异常处理

进行网络请求的时候，难免会出错，这时候为了程序能够继续运行下去，就要对异常进行处理

```python
# 最简单的一种

try:
    # 请求过程
    r = requests.get(url1, headers=headers, verify=False, params=data, cookies=cookies)
    # ......
    
except Exception as e:
    time.sleep(1)
    print(e)
    
finally:
    print("程序执行结束!")
    
```





###### 文件上传

```python
import requests

files = {
    'file':open('favicon.ico','rb')
}

r = requests.post('http://www.httpbin.org/post',files=files)
print(r.text)


```





































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python%E4%B9%8Brequests%E5%BA%93/  

