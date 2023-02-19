# Java学习之网络编程初涉


### 网络编程概述

Java提供的网络类库，可以实现无痛的网络连接，联网的底层细节被l隐藏在 Java的本机安装系统里，由JVM进行控制。并且Java实现了一个跨平台的网络库，**程序员面对的是一个统一的网络编程环境。**



**计算机网络:**
把分布在不同地理区域的计算机与专门的外部设备用通信线路互连成一个规模大、功能强的网络系统，从而使众多的计算机可以方便地互相传递信息、共享硬件、软件、数据信息等资源。



**网络编程的目的:**
直接或间接地通过网络协议与其它计算机实现数据交换，进行通讯。



**网络编程中有两个主要的问题:**

- 如何准确地定位网络上一台或多台主机; 定位主机上的特定的应用
- 找到主机后如何可靠高效地进行数据传输



**网络模型：**

- OSI参考模型
- TCP/IP参考模型
- 五层模型







### 网络通信要素概述

1. IP和端口号
2. 网络通信协议



#### 通信要素1：IP和端口号



##### IP:

> Java使用InetAddress类代替IP

唯一的标识Internet上的计算机（通信实体)
本地回环地址(hostAddress):127.0.0.1 主机名(hostName): localhost

**IP地址分类方式1:IPV4和IPV6**
IPV4:4个字节组成，4个0-255。大概42亿，30亿都在北美，亚洲4亿。2011年初已经用尽。以点分十进制表示，如192.168.0.1

IPV6:128位（16个字节），写成8个无符号整数，每个整数用四个十六进制位表示，数之间用冒号(:）分开，如:3ffe:3201:1401:1280:c8ff:fe4d:db39:1984

**IP地址分类方式2:** 公网地址(万维网使用)和私有地址(局域网使用)。

|      | 公网地址                        | 私有地址                                                   |
| ---- | ------------------------------- | ---------------------------------------------------------- |
| A    | 1.0.0.0 -127.0.0.0              | 10.0.0.0/8  ---->  10.0.0.0~10.255.255.255（A类）          |
| B    | 128.0.0.0-191.255.255.255       | 172.16.0.0/12  ----> 172.16.0.0~172.31.255.255（B类）      |
| C    | 192.0.0.0-223.255.255.255       | 192.168.0.0/16   ---->  192.168.0.0~192.168.255.255（C类） |
| D    | 240.0.0.0-255.255.255.254(多播) |                                                            |



##### 端口号：

> 端口号标识正在计算机上运行的进程（程序）

不同的进程有不同的端口号
被规定为一个16位的整数0~65535。

**端口分类:**

- **公认端口:** 0~1023。被预先定义的服务通信占用（如:HTTP占用端口8o，FTP占用端口21，TeInet占用端口23)
- **注册端口:** 1024~49151。分配给用户进程或应用程序。(如:Tomcat占用端口8080，MySQL占用端口3306，Oracle占用端口1521等）。
- **动态/私有端口:** 49152~65535。



**端口号与IP地址的组合得出一个网络套接字:Socket。**



```java
/* * 一、Java使用InetAddress类代替IP
 *
 *
 *
 *三、IP和端口号
 * 1.实例化InetAddress：两个方法：getByName(String host)： 获取ip、getLocalHost(): 获取本机dip
 *                  两个常用方法：getHostName(): 获取主机名称、getHostAddress() : 获取ip地址
 */

@Test
public void test() throws UnknownHostException {
    //实例化InetAddress
    // getByName(String host)： 获取ip
    // getLocalHost(): 获取本机dip
    InetAddress inet1 = InetAddress.getByName("192.168.0.112"); //ip
    InetAddress inet2 = InetAddress.getByName("www.sina.com"); //域名方式，获取ip
    InetAddress inet3 = InetAddress.getByName("127.0.0.1");
    InetAddress inet4 = InetAddress.getLocalHost();
    System.out.println(inet1);
    System.out.println(inet2);
    System.out.println(inet3);
    System.out.println(inet4);

    //getHostName(): 获取主机名称
    System.out.println(inet2.getHostName());
    //getHostAddress() : 获取ip地址
    System.out.println(inet2.getHostAddress());


}

```





#### 通信要素2：网络协议

**网络通信协议:**
计算机网络中实现通信必须有一些约定，即通信协议，对速率、传输代码、代码结构、传输控制步骤、出错控制等制定标准。

计算机网络通信涉及内容很多，比如指定源地址和目标地址，加密解密，压缩解压缩，差错控制，流量控制，路由控制，如何实现如此复杂的网络协议呢?通信协议分层的思想
在制定协议时，把复杂成份分解成一些简单的成份，再将它们复合起来。最常用的复合方式是层次方式，即同层间可吆通信、上一层可以调用下一层，而与再下一层不发生关系。各层互不影响，利于系统的开发和扩展。



**传输层协议中有两个非常重要的协议:**

- 传输控制协议TCP(Transmission Control Protocol)
- 用户数据报协议UDP(User Datagram Protocol)。
- TCP/IP以其两个主要协议: **传输控制协议(TCP)和网络互联协议(IP)**而得名，实际上是一组协议，包括多个具有不同功能且互为关联的协议。
- IP(Internet Protocol)协议是网络层的主要协议，支持网间互连的数据通信
- TCP/IP协议模型从更实用的角度出发，形成了高效的四层体系结构，即**物理链路层、IP层、传输层和应用层**。



**TCP协议:**

- 使用TCP协议前，须先建立TCP连接，形成传输数据通道
- 传输前，采用**“三次握手”**方式，点对点通信，是**可靠**的
- TCP协议进行通信的两个应用进程:客户端、服务端。
- 在连接中可进行**大数据量的传输**
- 传输完毕，需**释放已建立的连接，效率低**

**UDP协议:**

- 将数据、源、目的封装成数据包，**不需要建立连接**
- 每个数据报的大小限制在64K内
- 发送不管对方是否准备好，接收方收到也不确认，故是不可靠的
- 可以广播发送
- 发送数据结束时**无需释放资源，开销小，速度快**



![三次握手](https://image.geoer.cn/%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.jpg)



![四次挥手](https://image.geoer.cn/%E5%9B%9B%E6%AC%A1%E6%8C%A5%E6%89%8B.jpg)







### TCP网络编程

实例1：客户端发送信息给服务端，服务端将信息显示在控制台上

```java
  //客户端
    @Test
    public void client(){
        InetAddress inet = null;
        Socket socket = null;
        OutputStream os = null;
        try {
            //1.创建socket对象
            inet = InetAddress.getByName("127.0.0.1"); //设置服务端的ip
            socket = new Socket(inet, 8899);

            //2.获取输出流，用于输出数据
            os = socket.getOutputStream();
            //3.写出数据的操作
            os.write("你好，我是客户端mm".getBytes());
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //4.关闭
                try {
                    if(os != null){
                    os.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
                try {
                    if(socket != null){
                        socket.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
        }
    }



    //服务端
    @Test
    public void server(){
        ServerSocket ss = null;
        Socket socket = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;
        try{
            //1.创建服务器端的ServerSocket，指明自己的端口号
            ss = new ServerSocket(8899);
            //2.调用accept()表示接收来自于客户端的socket
            socket = ss.accept();
            //3.获取输入流
            is = socket.getInputStream();

//            不建议这么写：容易出现乱码
//            byte[] buffer = new byte[20];
//            int len;
//            while ((len = is.read(buffer)) != -1){
//                String str = new String(buffer,0, len);
//                System.out.println(str);
//            }
            //4.读取输入流中的数据
            //用ByteArrayOutputStream的形式来输出
            baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[5];
            int len;
            while ((len = is.read(buffer)) != -1){
                baos.write(buffer,0,len);  //写到内置的数组中了
            }
            System.out.println(baos.toString()); //把内部的数组输出成string

            System.out.println("收到来自客户端ip：" + socket.getInetAddress().getHostAddress() + "个数据");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            //5.关闭
            try {
                if(baos != null){
                    baos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if(is != null){
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if(socket != null){
                    socket.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if(ss != null){
                    ss.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```



实例2：客户端发送文件给服务端，服务端将文件保存在本地。

```java
//客户端2
@Test
public void client2(){
    InetAddress inet = null;
    Socket socket = null;
    OutputStream os = null;
    FileInputStream fis = null;
    try {
        //1.创建socket对象
        inet = InetAddress.getByName("127.0.0.1"); //设置服务端的ip
        socket = new Socket(inet, 8899);

        //2.获取流
        os = socket.getOutputStream();
        fis = new FileInputStream(new File("d:\\io\\1.png"));
        //3.写出数据的操作
        byte[] buffer = new byte[1024];
        int len;
        while ((len = fis.read(buffer)) != -1){
            os.write(buffer,0,len);
        }

    } catch (Exception e) {
        e.printStackTrace();
    }finally {
        //4.关闭
        try {
            if(fis != null){
                fis.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(os != null){
                os.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(socket != null){
                socket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

}

//服务端2
@Test
public void server2(){
    ServerSocket ss = null;
    Socket socket = null;
    InputStream is = null;
    FileOutputStream fos = null;
    try{
        //1.创建服务器端的ServerSocket，指明自己的端口号
        ss = new ServerSocket(8899);
        //2.调用accept()表示接收来自于客户端的socket
        socket = ss.accept();
        //3.获取输入流
        is = socket.getInputStream();
        fos = new FileOutputStream(new File("d:\\io\\1_out.png"));

        //3.写出数据的操作
        byte[] buffer = new byte[1024];
        int len;
        while ((len = is.read(buffer)) != -1){
            fos.write(buffer,0,len); //写入文件数据到本地
        }

        System.out.println("收到来自客户端ip：" + socket.getInetAddress().getHostAddress() + "个数据");
    }catch (Exception e){
        e.printStackTrace();
    }finally {
        //5.关闭
        try {
            if(fos != null){
                fos.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(is != null){
                is.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(socket != null){
                socket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(ss != null){
                ss.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}


```



实例3：从客户端发送文件给服务端，服务端保存到本地。并返回“发送成功”给客户端。并关闭相应的连接。

```java
//客户端3
@Test
public void client3(){
    InetAddress inet = null;
    Socket socket = null;
    OutputStream os = null;
    FileInputStream fis = null;
    InputStream is = null;
    ByteArrayOutputStream baos = null;
    try {
        //1.创建socket对象
        inet = InetAddress.getByName("127.0.0.1"); //设置服务端的ip
        socket = new Socket(inet, 8899);

        //2.获取流
        os = socket.getOutputStream();
        fis = new FileInputStream(new File("d:\\io\\1.png"));
        //3.写出数据的操作
        byte[] buffer = new byte[1024];
        int len;
        while ((len = fis.read(buffer)) != -1){
            os.write(buffer,0,len);
        }

        //客户端关闭数据的输出
        socket.shutdownOutput();

        //5.接收来自服务端的消息，并显示到终端
        is = socket.getInputStream();
        baos = new ByteArrayOutputStream();
        byte[] buffer2 = new byte[5];
        int len2;
        while ((len2 = is.read(buffer2)) != -1){
            baos.write(buffer2,0,len2);  //写到内置的数组中了
        }
        System.out.println(baos.toString()); //把内部的数组输出成string



    } catch (Exception e) {
        e.printStackTrace();
    }finally {
        //6.关闭
        try {
            if(fis != null){
                fis.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(os != null){
                os.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(is != null){
                is.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(socket != null){
                socket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(baos != null){
                baos.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }


    }

}

//服务端3
@Test
public void server3(){
    ServerSocket ss = null;
    Socket socket = null;
    InputStream is = null;
    FileOutputStream fos = null;
    OutputStream os = null;
    try{
        //1.创建服务器端的ServerSocket，指明自己的端口号
        ss = new ServerSocket(8899);
        //2.调用accept()表示接收来自于客户端的socket
        socket = ss.accept();
        //3.获取输入流
        is = socket.getInputStream();
        fos = new FileOutputStream(new File("d:\\io\\1_out.png"));

        //3.写出数据的操作
        byte[] buffer = new byte[1024];
        int len;
        while ((len = is.read(buffer)) != -1){
            fos.write(buffer,0,len); //写入文件数据到本地
        }
        System.out.println("图片传输完成!");

        System.out.println("收到来自客户端ip：" + socket.getInetAddress().getHostAddress() + "个数据");

        //4.给客户端反馈
        os = socket.getOutputStream();
        os.write("你好，照片已收到".getBytes());

    }catch (Exception e){
        e.printStackTrace();
    }finally {
        //5.关闭
        try {
            if(fos != null){
                fos.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(os != null){
                os.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(is != null){
                is.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(socket != null){
                socket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(ss != null){
                ss.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}


```





### UDP网络编程

- 类`DatagramSocket`和 `DatagramPacket`实现了基于UDP 协议网络程序。
- UDP数据报通过数据报套接字DatagramSocket发送和接收，**系统不保证UDP数据报一定能够安全送到目的地，也不能确定什么时候可以抵达**。
- DatagramPacket对象封装了UDP数据报，在数据报中包含了发送端的IP地址和端口号以及接收端的IP地址和端口号。
- UDP协议中每个数据报都给出了完整的地址信息，因此无须建立发送方和接收方的连接。如同发快递包裹一样。



```java
package cn.xpshuai.java1;

import org.junit.Test;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-31 11:54
 * @功能：UDP网络编程
 *
 *
 * 例子：
 *
 */
public class UDPTest {
    //发送端
    @Test
    public void send() throws IOException {
        DatagramSocket socket = new DatagramSocket();

        String str = "我是UDP数据包的大导弹";
        byte[] data = str.getBytes();
        InetAddress inet = InetAddress.getLocalHost();
        DatagramPacket packet = new DatagramPacket(data,0,data.length,inet, 9090);

        socket.send(packet);

        socket.close();
    }
    

    //接收端
    @Test
    public void receive() throws IOException{
        DatagramSocket socket = new DatagramSocket(9090); //指定端口号

        byte[] buffer = new byte[1024];
        DatagramPacket packet = new DatagramPacket(buffer,0,buffer.length);
        socket.receive(packet);

        System.out.println(new String(packet.getData(), 0, packet.getLength()));
    
        socket.close();
    }
}

```







### URL编程

#### 概述

- **URL(Uniform Resource Locator)**: 统一资源定位符，它表示Internet上**某一资源**的地址。
- 它是一种具体的UR，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。
- 通过URL我们可以访问Internet上的各种网络资源，比如最常见的 www，ftp站点。浏览器通过解析给定的URL可以在网络上查找相应的文件或其他资源。
- **URL的基本结构由5部分组成:**
  `<传输协议>:<主机名>:<端口号>/<文件名>#片段名?参数列表`
- 例如:
    http://192.168.1.100:8080/helloworld/index.jsp#a?username=shkstart&password=123>
- 片段名: 即**锚点**，例如看小说，直接定位到章节
- 参数列表格式: `参数名=参数值&参数名=参数值....`





#### URL类构造器

![url类构造器](https://image.geoer.cn/java-url%E7%B1%BB%E6%9E%84%E9%80%A0%E5%99%A8.jpg)



#### URL类的方法

一个URL对象生成后，其属性是不能被改变的，但可以通过它给定的方法来获取这些属性:

```java
public string getProtocol()	//获取该URL的协议名
public String getHost( )	//获取该URL的主机名
public String getPort( )	//获取该URL的端口号
public String getPath( )	//获取该URL的文件路径
public String getfile( )	//获取该URL的文件名
public String getQuery( )	//获取该URL的查询名

```

```java
try{
    URL url = new URL("http://www.xpshuai.cn/posts/29903/#toc-heading-8");
    System.out.println(url.getProtocol());
    System.out.println(url.getHost());
    System.out.println(url.getPort());
    System.out.println(url.getPath());
    System.out.println(url.getFile());
    System.out.println(url.getQuery());

}catch (Exception e){
    e.printStackTrace();
}
```





**例子：**实现服务端数据下载

```java
//实现服务端数据下载
@Test
public void urlTest2() {
    HttpURLConnection urlConnection = null;
    InputStream is = null;
    FileOutputStream fos = null;
    try{
        URL url = new URL("http://www.xpshuai.cn/medias/featureimages/15.jpg");

        urlConnection = (HttpURLConnection) url.openConnection();
        //真正的去获取
        urlConnection.connect();

        is = urlConnection.getInputStream(); //获取到输入流
        fos = new FileOutputStream("d:\\io\\pic.jpg");

        byte[] buffer = new byte[1024];
        int len;
        while ((len = is.read(buffer)) != -1){
            fos.write(buffer, 0, len);
        }
    }catch (Exception e){
        e.printStackTrace();
    }finally {
        try {
            if(is != null){
                is.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(fos != null){
                fos.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        if (urlConnection != null) {
            urlConnection.disconnect();
        }
    }




}

```



























---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E5%88%9D%E6%B6%89/  

