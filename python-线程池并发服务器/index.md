# python-线程池并发服务器



  
server

```python

# 使用线程池来实现并发服务器

import socket
from multiprocessing.dummy import Pool


def worker(conn):
    while True:
        recv_data = conn.recv(1000)
        if  not recv_data:
            break
        print("客户端{}发送了{}".format(conn, recv_data.decode()))
        conn.send(recv_data)
    conn.close()


if __name__ == '__main__':
    server = socket.socket()
    server.bind(('127.0.0.1', 8888))
    server.listen(1000)
    pool = Pool(3)
    while True:
        conn, addr = server.accept()
        print("客户端{}连接成功".format(addr))
        pool.apply_async(func=worker, args=(conn,))

```


client
  
```python
import socket


c = socket.socket()

c.connect(('127.0.0.1',8888))

while True:
    msg = input('>>>')
    if msg:
        c.send(msg.encode())   # 只能发送 bytes 类型的数据   encode将中文的变成byte的
        print(c.recv(1024))
    else:
        break

c.close()

```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/python-%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%B9%B6%E5%8F%91%E6%9C%8D%E5%8A%A1%E5%99%A8/  

