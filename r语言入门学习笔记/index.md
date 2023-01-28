# R语言入门学习篇一






## R语言起源

R语言由新西兰奥克兰大学Ross Ihaka和Ribert Gentlenman两人共同发明，其词法和语法分别源自Scheme和S语言。

R定义：一个能够自由有效地用于统计计算和绘图的语言和环境，它提供了广泛的统计分析和绘图技术。



## R语言的优势

- 免费开源
- 全面的统计研究平台，提供各种各专业的数据分析技术
- 是一个程序设计语言，所以他的能力很容易通过使用用户定义的函数扩展
- 拥有顶尖水准的制图功能
- 开源从多个数据源获取数据并将其转换为可用的形式
- 可运行在多种平台上



## R的学习资源

- R的主页 ：http://www.r-project.org
- CRAN：http://cran.r-project.org
- R的博客(推荐多逛逛)：http://www.r-bloggers.com
- R的数据：《数据挖掘与R语言》、《R语言实战》







## R的安装

**下载：**

进入R的主页http://www.r-project.org，找到download，选择一个镜像进入，然后选择适合自己电脑的版本进行下载

**2.安装：**

默认安装即可



## R的基本使用/常用命令

> 变量是区分大小写的



r的赋值符号可以为`=`或者`<-`（更常用后者）

```R
a <- 3  # 把3赋值给a

b <<- 2  # 强制赋值给全局变量b

s <- sum(1,2,3,4)
m <- mean(1,2,3,4)
m1 <- max(1,5,2,3)

```



**查看和删除对象：**

```R
# 查看
ls()  #查看当前工作空间中已经使用的变量、函数等对象
ls.str() #列出全部及详细信息
ls(all.names = TRUE) #查看包含隐藏变量、函数等的全部对象

# 查看变量长度
length(a)

# 查看变量类型
mode(v)

# 删除对象
# 常量、变量、函数等都可以称为对象
rm(a)  #删除一个对象a(以节约空间)
rm(b,c) #删除多个对象b,c,...
rm(list = ls())  #删除所有对象


#查看命令com的变量格式
args(com)

```



向量的操作：

```R

# 求q中所有元素分别的平方根
y <- sqrt(a)

z <- a + a2  # 元素相加，长度不一致--->使得较短的向量不断重复循环


# 生成1到1000的向量
x <- 1:1000

# 生成序列,从1到20增量为3
seq(1,20,3)

# 生成向量，值为5，循环10次
rep(5,15)
rep(1:3, 3) # 1-3的序列，循环三次

x = rep(v, each = n)
#依次对向量v的每个元素复制n此生成新的向量x。


# 生成10个服从均值为0，标准差为1的向量
rnorm(10)

# 生成服从正态分布的6个均值为5，标准差为2的向量
rnorm(6, mean=5, sd=2)

# 取出大于0的
x <- c(0, -3, 4, -1, 45, 98, 12)
x[x>0]
x[x>=2 | x<5]  # 取小于5，大于等于2的
x[-5] # 不取第五个值
x[-(1:3)]  # 不取第1-3元素





x = round(v)
#生成一个向量x，其中每个元素是v对应元素的最近整数。
 
> order(x)
#获得向量x第i大元素在向量中的位置。
 
> rank(x)
#获得向量x每个元素大小位置。
 
> sort(x)
#对向量x从小到大进行排序。降序：sort(x, decreasing = TRUE)。
 
> tapply(x,f,g)
#根据因子f对向量x分类执行函数g。
 
> split(x,f)
#向量x按因子f分类。
 
> diff(x)
#返回向量x的差分向量。
 
> cumsum(x)
#返回向量x的累加向量。

```





**查看历史命令：**

```R
# 查看敲过的命令（默认保存在当前系统用户的.history文件里，我们也可以设置保存的其文件）
history

history(10)  #显示最近执行的10条命令
#请空Console窗口：ctrl+L
```





与逻辑型数据有关的基本操作:

```R
 is.data.frame(x)
#判断是否对象x是数据框。类似命令有is.ts(x)，is.numeric(x)等。
 
> all(x>a)
#判断是否对象x的每个元素都大于a。
 
> any(x>a)
#判断对象x的元素中是否存在一个大于a。
 
> x>y
#判断x的每个元素是否大于y的每个元素。
 
> x[x>a]
#向量x中大于a的元素组成的新向量。
 
> subset(x, x>a)
#向量x中大于a的元素组成的新向量。与上面例子的区别在于若向量元素里有NA，上面的例子会保留在结果中，而subset命令会剔除掉。
 
> which(x, x>a)
#返回向量中大于a的元素的位置。
 
> x = ifelse(b, u, v)
#生成一个与b(逻辑向量)维度相同的数值向量，若b[i]为TRUE，则x[i]为u，反之为v
```





**一个简单例子：**

![](http://image.xpshuai.cn/%E8%AE%A4%E8%AF%86R%E7%9A%84%E5%B0%8F%E4%BE%8B%E5%AD%901.png)

```R
age <- c(1,3,5,2,11,9,3,9,12,3)
weight <- c(4.4,5.3,7.2,5.2,8.5,7.3,6.0,10.4,10.2,6.1)

#mean求平均值,sd求标准差
mean(weight)
sd(weight)
#查看关系，可以得到相关度
cor(age,weight)
#也可以用plot画图来说明
plot(age,weight)

# 查看R可以画的图
demo()
demo(graphics)
```



**查看帮助文档：**

```R
# 查看帮助文档
help.start()  # 打开帮助文档的首页
help("mean")	# 查看某一命令的用法，mean{base}意思是mean属于base包
help(package="base")  # 查看包的信息
?mean   # 同上
```





**R的工作空间：是当前所有的变量都存放的位置

> 建议养成习惯：给每个项目单独设置一个工作空间

```R
# 查看工作空间
getwd()

# 设置新的工作空间
setwd(dir="d://RProject/test1")

list.files()  #查看当前工作目录下的文件
dir()         #查看当前工作目录下的文件


save.image()  #保存工作空间，防止断电等外部因素
```







##  R包的使用



R的包(package)可以从：http://cran.r.project.org/web/packages下载。

R自带了一系列默认包（base、datasets、grephics、methods等）



**包的使用：**

```R
#列出已安装的包。
library()
 
# 安装新的包
install.packages("car")

# 查看包的信息
help(package="car") 


# 使用包--->需要先再入，有下面两种方式
library(ggplot)  
require(ggplot)

# 更新包
update.packages()




library(help = AER)
#获取包AER的信息。
 
detach(package:zoo)
#去除载入的包zoo。
 
search(reshape)
#列出已载入的包。

```



一般对象的基本操作：

```R
objects()
ls()
#列出所有对象


mode(x)
#查看对象x的模式：空，数值，字符，逻辑，复数，列表，函数(NULL，numeric，character，logical，complex，list，function)。



class(x)
#查看对象x的类型：除了mode里列出的几种类型外，还有整数，矩阵，因子，阵列，数据框，时间序列(integer，matrix，factor，array，data frame，ts)等其他类型。mode主要用于区别数据存放的方式，而class是一种更细微的分类方式，比如矩阵，就是一种更“有序”的数据存放方式。此命令比mode常用


> as.matrix(x)
#把对象x转为矩阵型。
 
> as.numeric(x)
#把对象x转为数值型。
 
>as.factor(x)
#把对象x转化为因子型。
 
> str(x)
#查看对象x的结构。str是structure的缩写。
 
> rm(x)
#移除对象x。
 
> rm(list=ls(all=TRUE))
#移除所有对象。
 
>head(x)
#查看数据的前x行

```







结果的重用：

```R
#如下场景
# head 查看
head(mtcars)
# lm是线性回归
lm(mpg~wt, data=mtcars)  

# 我们通常使用变量来保存,实现了重用：
result <- lm(mpg~wt, data=mtcars) 
# summary查看
summary(result)
#然后就可以用来进一步分析和使用了

# predict预测回归的值
predict(result, mynewdata)
```



**R如何处理大数据集：**

r的运算都是基于内存的，但是对于很大的数据运算显然是不合适的：

- R有专门用于大数据的分析报告，如lm()是做线性拟合的函数，而biglm()则能以内存搞笑的方式实现大型数据的线性模型拟合
- R与大数据处理平台的结合，如RHadoop、RHive、RHipe等





## 数据集

按照某种格式来**创建数据**集，是任何数据分析的第一步：

1. 选择一种数据结构来存储
2. 将数据输入或导入这个数据结构中



向R中导入数据有很多方便的方法，可以手工输入数据，也可以从外部源导入数据，数据源可以是电子表格、文本文件、统计软件(SAS)和各类数据库等



**数据集**通常由数据构成的一个矩形数组，行表示记录，列表示属性(字段)。



R拥有许多用于存储数据的对象类型，包括向量、矩阵、数组、数据框和列表，这些数据结构在存储数据的类型、创建方式、定位和访问我中个别元素的方法等方面都有所不同。

列表：向量、矩阵、数组、数据库、列表



### 向量：

只能包括一种数据类型

```R
# 向量
a <- c(1,3,5,7,-5)
a2 <- c("one", "two", "three")
a3 <- c(1,2,6,"three")  # 都强制转换变成字符串了

# 访问向量中元素
a2[2]
a[1:3]
a[c(1,3,4)]

# 修改元素值（R语言可以对变量中每一个元素直接进行操作）
a2[2] <- "four"

# 求q中所有元素分别的平方根
y <- sqrt(a)

z <- a + a2  # 元素相加，长度不一致--->使得较短的向量不断重复循环


# 生成1到1000的向量
x <- 1:1000

# 生成序列,从1到20增量为3
seq(1,20,3)
```



### 矩阵：

只能包括一种数据类型，二维的

```R
?matrix  # 查看矩阵的用法

## 创建矩阵
y = matrix(5:24,nrow=4,ncol=5) # 默认按列填充


## 创建矩阵
x=c(2,4,6,8)
rowname <- c("R1", "R2")
colname <- c("C1", "C2")

newMymatrix <- matrix(x, nrow=2,ncol=2,byrow=TRUE,dimnames=list(rowname,colname)) # 按行填充

newMymatrix2 <- matrix(x, nrow=2,ncol=2,dimnames=list(rowname,colname)) # 按列填充
# 通过下标进行元素选择
newMymatrix2[1,]    # 访问第一行
newMymatrix2[1,3]   # 访问第一行第三个元素
newMymatrix2[,3]   # 访问三列


M = rbind(X,Y)
#按行合并矩阵X和Y形成新矩阵M。(X和Y列数需相同）
 
M = cbind(X,Y)
#按列合并矩阵X和Y形成新矩阵M。(X和Y行数需相同）
 

diag(M)
#矩阵M的对角线元素形成的向量


#求矩阵M的特征值和特征向量。
eigen(M)$val
eigen(M)$vec
 



```



![创建矩阵](http://image.xpshuai.cn/martrix_create.png)





### 数组：

维度是可以大于2的，元素也只能是一种类型的

```R
?array
# 先确定维度的名字，3x2x4的
dim1 <- c("A1", "A2", "A3")
dim2 <- c("B1", "B2")
dim3 <- c("C1", "C2", "C3", "C4")

# 创建数组，参数：数据,维度,维度名字
d <- array(1:24, c(3,2,4), dimname=list(dim1.dim2,dim3))

#选取元素
d[1,2,3]  # 选取第一行，第二列，第三个
```





### 数据框

> 可以存储多种类型的数据

数据框会是我们以后更经常遇到的一种结构



```R
# 创建数据框   data <- data.frame(col1, col2, ...)

#例子：
id <- c(1,2,3,4)
age <- c(25,34,28,52)
itype <- c("Type1","Type2","Type3","Type2")
status <- c("poor", "Improved", "Excellent", "poor")

#下面 创建病例数据的数据框
patientsData <- dada.frame(id, age, itype, status)


#取数据
patientsData[1:2]  # 取第一列和第二列
patientsData[c("itype", "status")] # 根据列名取值
patientsData$age	# 使用$来取值
# attach函数将数据框加入搜索路径，以后直接输入列名来取值
attach(mtcars)
mgp
detach(mtchar)	# 从搜索路径移出
# with()函数功能同attach
with(mtcars, {
  ll <- mpg
  ll
})


fix(Data)
#编辑数据框Data




```







### 因子

```R
itype <- c("Type1","Type2","Type3","Type2")

# factor将factor以数组形式存储，将Type1和1联系起来，将Type2和2联系起来
itype <- factor(itype)

```

待查资料

https://www.runoob.com/r/r-factor.html





### 列表

是较复杂的数据类型



```R
# myLi <- list(object1, object2, ...)

g <- "First list"
h <- c(23,45,66,997)
j <- matrix(1:10, nrow=2)
k <- c("one", "two", "three")
# 创建列表
myList <- list(g, h, j, k)  # 将前面4个对象将入list


# 访问list中元素（和其他数据类型有区别）
myList[[2]] # 用双重括号。这里表示取第二个对象



```



```R
# 创建一个列表
myLi <- list(stu.id=1234,
            stud.name="Tom",
            stud.marks="12,3,14,25,66")


# 获取第一个元素 [[]] 获取值
myLi[[1]]
mode(myLi[[1]])

# [] 单括号 获取第一个成分的"键值",即子列表
myLi[1]
mode(myLi[1])

myLi$stud.id

# names   获取list中的“键”
names(myLi)
names(myLi) <- c("idid", "name", "marks") # 可以修改
#去掉列表L里的对象名
unname(L)


# 扩充元素
myLi$parents <- c("Mom1", "Baba1")
length(myLi)


# 删减成分
myLi <- myLi[-4]  # 删除第四个成分

# 合并列表
other <- list(age=19, gender="male")
lst <- c(myLi, other)


# unlist变列表中元素为向量的形式  
unlist(lst)  # 每个向量都有一个名称



```









## R的数据源导入的方法

### 可导入的数据源：





#### 1.键盘输入



```r
#(实现保存)。不使用edit的话只是一个中间变量，关闭后会消失
# 使用edit
mydata <- edit(mydata)


# 使用fix（更简洁）
fix(mydata)

```





#### 2.从文本文件导入

> r默认只能读取ascii编码的(ANSI)，utf8等会有问题

文本文件的格式每行字段间用逗号隔开的

使用`read.table()`读取

```R
# 参数：txt位置，header为True会吧文件首行取下来作为标签，sep指定间隔符
data <- read.table("f:/data/q.txt",header=True,sep=",") # 路径不要有中文


head(data) # 查看数据
```



#### 3.导入Excel数据

导入excel数据方式很多(如)，但比较复杂，这里选择简便的方式：另存excel文件类型选择`csv`，并以逗号分隔，再`read.csv()`



> 目的是读入数据的结果，不在乎手段，所以越简单越好





#### 4. ......





## 用户自定义的函数

**格式：**

```r
myFUnction <- function(arg1, arg2, ...){
  statements
  return(obj)
}
```



实例1：获取系统时间，指定格式并输入

```R
# 定义了一个函数
# switch，如果形参type值为long，执行对应部分
mydate <- function(type){
  switch(type,	
        long = format(Sys.time(), "%A %B %d %Y"),
        short = format(Sys.time(), "%m-%d-%y"),
        cat(type, "is not reccongnized type\n")
        )
}

# 调用
mydate("long")
mydate("short")

```



实例2：求1....n的和

```R
sum <- function(num){
  
  for(i in 1:num){
    x <- x + i
  }
  return(x)
}

fix(sum) # 会进入编辑器中，打开后给x赋初值：在for循环前 x <- 0

# 调用
sum(10)

```





## R访问MySQL数据库

**准备工作步骤：**

1. 安装RODBC包：`install.packages("RODBC")`
2. 在http://dev.mysql.com/downloads/connector/odbc 下载connectors ODBC（选择适合自己系统的），然后安装
3. windows：控制面板->管理工具->数据源(ODBC)->双击->添加->选中mysql ODBC driver-->输入数据库信息



**使用：**

```R
#加载库
library(RODBC)
myconn <- odbcConnect("RData", uid="root", pwd="123456") # 数据库名，用户名，密码

# 导入mysql数据到r中
data1 <- sqlFetch(myconn, "news") # 指定数据表

# 导入查询到的数据到r中
data2 <- sqlQuery(myconn, "select id,title,pubtime feom news")

# 关闭连接
close(myconn)

```





## R的集成开发环境(IDE) - RStudio

RStudio使用C++开发，界面更加友好，功能更加强大。

我们先下载R，然后去官网下载免费版RStudio即可。





**使用：**



ctrl+s保存为`.R`结尾的文件



**run：**

![](http://image.xpshuai.cn/rstudio%20run.png)



**画图：**

会在右下角plot显示，还可以对图进行导出保存



**History：**

![](http://image.xpshuai.cn/rstudio1.png)



**R程序和默认工作空间：**

Tools-->option-->General-> R version 和Default working directory





**导入/保存工作空间：**

Session -> load / save workspace





**新建/打开工程：**

Project下面



**导入数据集：**

Tools --> Import Dataset













## 如何画图

### 简单的图画例子

数据集：

![image-20220412211923345](http://image.xpshuai.cn/img/image-20220412211923345.png)

画图：

```R
dose = c(20,30,40,45,60)
drugA = c(16,20,27,40,60)
drugB = c(15,18,25,31,40)
plot(dose, drugA, type = "b")  #b表示既绘制点也绘制线

```

![plot](http://image.xpshuai.cn/img/image-20220412214338899.png)

### 图形参数的修改

#### 线条、符号、颜色：

```r
# par
opar = par(no.readonly=TRUE)  # 生成的图形就可以进行修改了
# 进行修改
par(lty=2, pch=17)  # 虚线和三角形
# 再继续画图
plot(dose, drugA, type = "b") 
# 恢复最开始修改之前的图形
par(opar)

# 指定符号和线条，使用较多
plot(dose, drugA, type = "b", lty=6, pch=19, lwd=3, cex=2) 
help(plot) # 但并不是全部绘图函数都支持这个参数，可以先查看绘图函数都支持啥参数

# 指定颜色 colors()返回所有颜色的名称
#col = "#FFFFF"
#col = "red"
col = rgb(25,155,60)
...
plot(dose, drugA, type = "b", lty=6, pch=19, col="blue", col.axis="red", col.lab="green") 

```

![符号和线的参数](http://image.xpshuai.cn/img/image-20220412214851860.png)





![颜色的参数](http://image.xpshuai.cn/img/image-20220412215247676.png)







#### 文本属性

![文本相关参数](http://image.xpshuai.cn/img/image-20220413093750180.png)

```R
dose = c(20,30,40,45,60)
drugA = c(16,20,27,40,60)
drugB = c(15,18,25,31,40)

#opar = par(no.readonly=TRUE)  
#par(lty=2, pch=17)

plot(dose, drugA, type = "b", font.lab=3, cex.lab=1.5, font.main=2)    # 标题的样式为2倍，字体为斜体，

# 查看pdf能用的字体
names(pdfFonts())

```







#### 图形、边界尺寸



![图形、边界尺寸参数](http://image.xpshuai.cn/img/image-20220413094218857.png)

```R
#opar = par(no.readonly=TRUE)  
#par(pin=c(4,3), mai=c(1,.5,1,.2))  # 4英寸宽，3英寸高；边界为下1左0.5上1右0.2英寸

#plot(dose, drugA, type = "b", lty=2, col="red",bg="green") 

```



```R
# 添加标题和坐标轴
plot(dose, drugA, type = "b", lty=2, col="red",bg="green", main="药物A的反应曲线", sub="一个测试数据", xlab="剂量",ylab="病人的反应",xlim=c(0,60),ylim=c(0,70))  # xlim是横坐标的刻度


# 或者用title 函数来调价标题
title(main="我的标题", col.main="red")
title(sub="我的副标题", col.sub="green")
```



#### 自定义坐标轴

![坐标轴参数](http://image.xpshuai.cn/img/image-20220413094218857.png)

```R
#plot(dose, drugA, type = "b",pch=12,col="red",mar=c(5,4,4,8)+0.1, yaxt="n", ann=FALSE) # mar是边界,  yaxt="n"禁用y轴刻度, ann=FALSE不会出现坐标轴默认的描述# 设置自己的参数

y = c(1,10)
x = y
z =  10/x

opar = par(no.readonly=TRUE)
par(mar=c(5,4,4,8)+0.1)
plot(x,y, type = "b",pch=12,col="red", yaxt="n", lty=3,ann=FALSE)

axis(2, at=y, labels=x, col.axis="blue", las=2) # 2表示在左边添加刻度, las=2表示垂直与坐标轴,at表示刻度线的位置，col.axis坐标轴颜色
# lines和plot差不多，不过是在原来图像上画一条线
lines(x,z,type = "b",pch=12,col="green", lty=2) # 4表示在右边添加刻度
axis(4, at=z,labels=round(z,digits=2), col.axis="black", las=2,cex.axis=.7)



```





#### 次要刻度线



```R
# 1.先下载并导入包
install.packages("Hmisc")
library(Hmisc)


# 使用
#以下面数据为例
plot(1:4, 1:4, type="b")
minor.tick(nx=2,ny=3,tick.ratio=0.5) # x轴分为几段，y轴分为几段，强度


```



#### 参考线

```R
# abline(h=value, v=value) h为水平线，v为垂直线
abline(h=2, col="red", lty=2) # 在水平第二个位置添加水平线
abline(v=2.5, col="red", lty=1) 

```



#### 图例

![图例](http://image.xpshuai.cn/img/image-20220413105334159.png)





```R
opar = par(no.readonly=TRUE)
par(lwd=2, cex=1.5, font.lab=2)
#药物A的
plot(dose, drugA, type="b",pch=15,col="red",lty=1, ylim=c(0,60), main="药物A和药物B的对比", xlab="剂量", ylab="药物反应")

#药物B的，在原来图的基上画
lines(dose, drugB, type="b",pch=17,col="blue",lty=2)

#更改刻度
minor.tick(nx=5,ny=2,tick.ratio=0.5)

#图例
legend("topleft", inset=.05, title="类型", legend("A","B"), lty=c(1,2),pch=c(15,17),col=c("red","blue")) # 顺序要和前面一样,lty是线的类型,pch是点符号的类型,col是颜色

```





#### 文本标注

> 想针对某个点进行标注的时候

![文本标注](http://image.xpshuai.cn/img/image-20220413111139687.png)



```R
attach(mtcars)
plot(wt,mpg, main="车重和油耗关系", xlab="车重", xlab="油耗", pth=18,col="blue")

text(wt,mpg,row.names(mcars),cex=0.5, pos=4, col="red")
```





#### 图形组合、图形布局的精细控制

![R如何画图](http://image.xpshuai.cn/img/image-20220413112126712.png)

**图形组合：**

```R
attach(mtcars)
opar <- par(no.readonly=TRUE)
# 设置为两行两列
par(mfrow=c(2,2)) 
#画第1个图
plot(wt,mpg,main="wt vs mpg")
#画第2个图
plot(wt,disp,main="wt vs disp")
#画第3个图
hist(wt,main="Histogram of wt")
#画第4个图
boxplot(wt,main="Boxplot of wt")
par(opar)
detach(mtcars)


# layout
# 两行两列，1,1,2,3第一行第一个和第二个是第一个图形，第二行第一列是第二个图形，第二行第二列是第三个图（相当于把矩阵的数字排列进去）
layout(martrix(c(1,1,2,3)), 2,2, byrow=TRUE)


# 精确控制每幅图像大小，使用width和和height.控制每列的宽度width=c(3,1)表示第一列占3/4
# 如果高度和宽度调整的不合适有可能图幅占不下显示不全

layout(martrix(c(1,1,2,3)), 2,2,width=c(3,1), height=c(1,2),byrow=TRUE)

```

![一个图幅画放多个图](http://image.xpshuai.cn/img/image-20220413112455452.png)



**图形精细控制：**



```R
par(fig(0, 0.8, 0, 0.8), new=TRUE)  # 画布左下角位置是0,0，右上角为0.8,0.8,  new=TRUE会在图形上继续添加

```

![画布坐标](http://image.xpshuai.cn/img/image-20220413114627765.png)







## R基本的数据管理

一个示例：

![示例1](http://image.xpshuai.cn/img/image-20220414210257598.png)



在数据框中创建新变量

```R
mydada <- data.frame(x1=c(2,3,4,5), x2=c(2,5,7,9))

#求和（和的一列会加入数据框）
mydata$sumx <- mydata$x1 + mydata$x2
#求平均值
mydata$meanx <- (mydata$x1 + mydata$x2)/2
```



变量的重编码（把连续的数字变成某字符串）

```R
#比如对上述例子的年龄进行重编码
#如：定义年龄为99的时候为空值
survey <- data.frame(......)
survey$age[survey$age == 99] <- Na

#如：定义年龄>75的时候定义为老年人
survey$age[survey$age > 75] <- "老年人"

#如：定义年龄<75并且>55的时候的时候定义为中年人
survey$age[survey$age > 55 & survey$age < 75] <- "中年人"


```

变量的重命名

```R
# 可以通过fix 调用可视化界面修改
fix(survey)

# 也可以使用names()
names(survey)[6] <- "question1" # 数据框[第几个值]
```





处理缺失值(NA)

```R
x <- c(1,2,NA)
# is.na() 判断是否为空，返回bool
is.na(x)
is.na(survey[,6:10]) # 判断第6-10列中是否有空值

# 不能使用 x == NA 这种方式判断

# 返回缺失值的位置
which(is.na(A))

#计算数据集A的缺失值总数
sum(is.na(A))

# 对NA值进行删除
y <- sum(x, na.rm=TRUE) # y返回x的和，对NA值进行删除

# 这种会删除空值所在的一整行:
data <- na.omit(survey)
```



日期值的使用

```r
# as.Date() 将字符串转换为日期

mydate <- as.Date("2022-04-14") # "2022-04-14"


mydate1 <- c("04/14/2022", "04/13/2022")
date <- as.Date(mydate1, "%m/%d/%Y") # 自定义显示格式


# Sys.Date()   返回系统日期

# date()   返回系统日期+时间

# format()  格式化输出
today <- Sys.Date()
format(today, format="%B %d %Y")  # 自定义格式化输出

# 天数减法
startdate <- as.Date("2022-01-02")
enddate <- as.Date("2022-04-15")
days <- enddate - startdate

```

![日期格式](http://image.xpshuai.cn/img/image-20220414212754868.png)





数据类型的判断及转换

```R
a <- c(2,5,7)
is.numeric(a)
is.vertor(a)

b <- as.character(a)
is.numeric(b)

```

![类型判断及转换函数](http://image.xpshuai.cn/img/image-20220414213710267.png)





数据排序

```R
# order() 函数

data <- survey[order(survey$age),] # 以survey的age来排序，默认升序, 加逗号表示对行排序，不加逗号表示对列排序！！！

# 降序，加负号
data <- survey[-order(survey$age),] 


data2 <- survey[order(survey$gender, survey$age),] # 先按性别排序，再按年龄排序


```





数据集合并

```R
# 列合并cbind()和merge()
x <- matrix(c(1,2,3,4,5,6,7,8,9), nrow=3, ncol=3)
y <- x
z <- cbind(x,y) # 合并x和y
# 或者用merge(根据字段k1进行连接，类似sql中的联合)
z1 <- merge(x,y, by="k1")



# 行合并  rbind()
# 前提：变量数目的列数必须相同


```





数据集子集的抽取

```R
# 方式1：
q <- survey[,6:10]  # 选取第6-10列

# 方式2：去除某部分
x <- survey[,-2] # 去掉第二列

# 按照条件选取，显示q1到q4列
newdata <- subset(surver, age>=35 | age<24, select=c(q1,q2,q3,q4))


```





随机抽样函数

```R

# sample()
sample(x, size, replace=FALSE) #  从x中抽取size个，replace=FALSE表示不放回抽样

mysample <- survey[sample(5,3,replace=FALSE),]


```









## R的高级数据管理

数学函数

```R
# abs()  求绝对值
# sqrt()  求平方根
# ceiling(x)  不小于x的最小整数
# floor(3.567)   不小于x的最大整数
# round(3.55)  四舍五入
# sin()  三角函数
# cos()
# log(10)  对数函数


```





统计函数

```R
x <- c(2,3,4,1,3,7,4,9,10)

mean(x)  # 求均值

sum(x)

# 求均值也可以这样
n <- sum(x)/length(x)

# 求标准差
sd(x)
# 求方差
var(x)

# 求最大最小值
min(x)
max(x)

# 标准化
scale(x)  # 默认对向量x使用均值为0标准差为1的

```





概率函数：

```R
x <- pretty(c(-3,3), 30)
y <- dnorm(x)  # 正态分布
plot(y)

# 随机正态分布函数：生成50个，均值为20，标准差为8的
rnorm(50，mean=20, sd=8)

# 生成一个服从正态分布的伪随机函数
runif(5)

# 
set.seed(12) # 先设定种子数
runif(5)

# 其他函数
```

![image-20220415093505576](http://image.xpshuai.cn/img/image-20220415093505576.png)





字符处理函数

```R
# nchar()  统计字符数量
nchar("dddf")

# 根据索引提取
substr("sahiu", 3,5)


# 查找（从向量中查找a）
grep("a", c("b","a","c","t"))


#字符串中a替换为A
sub("a","A", "abcde")

# strsplit()在字符串分割
strsplit("abcde", "c") # 以c分割
strsplit("abcde", "")  # 每个字符单独分开

# 字符串连接
paste("Thec", "Her is me.")

# 大小写转换
toupper("abc")
tolower("ABC")
```



其他实用函数

```R
length()	# 获取长度

rep(1:3, 5) # 将1到3重复5次

cat("hello") # 输出到屏幕上
```



将之前的函数应用于矩阵和数据框中：

```R
a <- c(1,2,4,5,6)
round(a) # 对所有值取


b <- matrix(runif(12), nrow=3)
log(b) # 针对全体进行对数
mean(b)

# apply()  将函数范围应用到数据库或数组某个维度
c <- matrix(rnorm(12), nrow=6, ncol=5) # 5列6行的服从正态的
apply(c ,1, mean) # 对c矩阵，按行就平均值
apply(c ,2, mean) # 对c矩阵，按列就平均值

# lapply    sapply  针对列表的单独操作

```



**重复和循环**

循环：

```R
# for
for(i in 1:5) print("hello")  # 循环1-5


# while
x <- 5
while(x > 0) {print("Hello"); x<- x-1} # 语句之间用分号

```

条件执行：

```R
# if-else
if(x != 1) print("male") else print("female")


# ifelse  满足条件执行1，不满足执行2
ifelse(条件, 语句1, 语句2)



# switch
felling <- c("sad", "glad", "afraid")
for(i in felling)
    print(switch(i, glad = "我很开心", afraid = "害怕", sad="悲伤")  # 如果i等于glad...


```



转置

```R
# T

cars <- mtcars[1:5, 1:4] # 取1-5行，1-4列
# 转置矩阵
t(cars)

```



其他内容课余时间多学





## R的基本图形



### 条形图

本节例子需要使用`vcd`包进行演示，`install.packages('vcd')`

#### 简单的条形图

barplot()

```R
library(vcd)

#barplot(height)  最简单的方式，指定高度，默认垂直
barplot(c(1,2,4,6,2,7))
barplot(c(1,2,4,6,2,7), horiz=TRUE) # 水平的图

# 示例数据：关节炎病人的治疗情况
#Arthritis
counts <- table(Arthritis$Improve)
barplot(counts)
```



#### 堆砌、分组条形图

> 绘制的数据不再仅仅是一个向量，而是矩阵

**堆砌条形图：**

```R
# barplot


counts <- table(Arthritis$Improve,Arthritis$Treatment) # 矩阵的形式
barplot(counts) # 默认是堆砌的条形图



```

![堆砌](http://image.xpshuai.cn/img/image-20220415140123913.png)

**分组条形图：**

```R
barplot(counts, beside=TRUE)  # beside=TRUE  值是并列的，不是堆砌的

```

![分组](http://image.xpshuai.cn/img/image-20220415140158858.png)



#### 均值条形图



```R

# 使用数据集 state.region,  state.x77
head(state)
states <- data.frame(state.region, state.x77)
x <- aggregate(states$Illiteracy, by=list(state.region), FUN=mean) # 统计地区的文盲率

barplot(x$x, names.arg=x$Group.1) # 画图并指定坐标名字
```

![均值](http://image.xpshuai.cn/img/image-20220415140706515.png)



#### 条形图的微调



```R
par(mar=c(5,8,4,2)) # 指定绘图区域范围
counts <- table(Arthritis$Improve,Arthritis$Treatment) 
par(las=2) # 标签由竖的旋转为横的

barplot(counts, horiz=TRUE, cex.names=0.8, names.arg=c("没有治疗的","一些治疗的", "显著治疗的")) 


```





### 饼图

**饼图：**

`pie()`

饼图缺陷：无法明确比较谁大谁小或相差多少

```R
# pie(x, label) # 参数1是数值向量，参数2是各个扇形标签的向量


par(mfrow=c(2,2)) # 画图区域分为四部分

x <- c(10,12,4,16,8)
lab <- c("国家1","国家2","国家3", "国家4", "国家5")
pct <- rouund(x/sum(x)*100)
labl <- c(lab, " ", "pct", %, sep=" ") # 自定义拼接制作标签

pie(x, labl, col=rainbow(length(labl)),main="简要饼图") # 指定名称, col=rainbow指定颜色


```

3D饼图

```R
# 借助包：plotrix
library(plotrix)

pie3D(x, labl, explode=0.1, main="3D饼图") # explode=0.1指定饼之间的裂隙

```



**扇图：** 扇图可以明确比较大小

```R
fan.plot(x, labels=lab, main="扇图")

```

![扇形](http://image.xpshuai.cn/img/image-20220415142252862.png)



### 直方图

柱条组成的，特点：x轴上将值域分成等分的组

`hist()`函数

```R

#hist(x)
x <- mtcars$mpg

# 最简单的直方图
hist(x)

```

![最简单的直方图](http://image.xpshuai.cn/img/image-20220415142725870.png)



```R
hist(x, breaks=12, col="red", xlab="每英里加仑数")
# x轴上划分为12组
```

![直方图2](http://image.xpshuai.cn/img/image-20220415142829530.png)



```R
#  freq=False则y轴上不显示频数而是概率密度
hist(x, freq=FALSE, breaks=12, col="green", xlab="每英里加仑数")

# 添加数值的噪声（如下图的小竖线）
rug(jitter(x))

# 添加轴虚线
lines(density(x), col="red", lwd=2) # lwd是线的宽度

```







### 核密度图

x轴展现是值，y轴是反映x轴值上密度的情况

>  可以很直观了解，比如不同条件下的变化情况

```R
# 先用density处理
x <- density(mtcars$mpg)
#  生成核密度图
plot(x) 

```

描述4缸或6缸汽车不同的耗油量的对比：

```R
attach(mtcars)

library(sm)
# 比较 sm.density.compare(x, factory)
sm.density.compare(mpg, cyl, xlab="英里每加仑")

```

![对比](http://image.xpshuai.cn/img/image-20220415144317890.png)







### 箱线图

**分位数**

![箱线](http://image.xpshuai.cn/img/image-20220415144446522.png)



**箱线图元素：**最小值、四分(之一)位数、中位数、四分(之三)为主、最大值、异常值





```R
# boxplot()

boxplot(mtcars$mpg, main="箱线图", ylab="公里每加仑")
# 
boxplot(mpg~cyl, data=mtcars, main="箱线图", ylab="公里每加仑", xlab="汽缸数")

# 可以看到6缸的是比较稳定的(上下窄)
# 做质量检测的时候经常用到
```

![箱线图](http://image.xpshuai.cn/img/image-20220415144928657.png)









## 实例分析--预测海藻数量

> 预测海藻数量



**1.问题描述与目标：**

![问题描述与目标](http://image.xpshuai.cn/img/image-20220415152642555.png)



**2.数据集的导入：**

![数据集的导入](http://image.xpshuai.cn/img/image-20220415153129276.png)

```R
# 以上数据集在 DMwR 包中
install.packages("DMwR2")
library("DMwR2")

# 先了解一下数据
head(algae)
summary(algae)


# 这里可以查看一下化学属性的图形
hist(algae$mxPH, prob=T, ylim=0:1) # prob表示显示概率密度
lines(density(algae$mxPH, na.rm=T))
```

![数据解读](http://image.xpshuai.cn/img/image-20220415160838980.png)





**3.数据预处理**

![数据缺失的处理](http://image.xpshuai.cn/img/image-20220415162038434.png)

查找缺失值：

```R
# 返回存在缺失值所在行的行数
algae[!complete.cases(algae), ]  

# 返回存在多个缺失值的位置
manyNAs(algae)
manyNAs(algae, 0.2) # 如果属性缺失超过所有缺失的20%就记录下来

```

缺失值的处理：

```R
# F1:将含有缺失值的记录删除
x <- algae
y <- na.omit()
y[!complete.case(y),]  # 可以发现没有缺失值了
x <- algae[-c(62,199),]  # 对于缺失值较多的情况，指定缺失值较多行来删掉
manyNAs(x)


# F2:根据变量之间相关关系填补缺失值
#首先要找数据间的相关关系
symnum(cor(algae[,4:18], use="complete.obs")) # 得到4-18字段之间的相关关系
x <- algae[-manyNAs(algae), ]
lm(PO4~oPO4)


# F3:根据案例之间相似性来填补
求距离值，把差值最小的依次排序，...

```



**4.获取预测模型**

![获取预测模型](http://image.xpshuai.cn/img/image-20220415165137648.png)





```R
x <- algae[-manyNAs(algae), ]

clean.algae <- knnImputation(algae,k=10) # 通过元素相似性的方法处理了缺失值
clean.algae

# 第一个模型
lm.a1 <- lm(a1 ~., data=clean.algae[, 1:12]) # a1 ~. 代表a1和所有其他数据字段建立的相关关系
summary(lm.a1)


```

![辅助变量](http://image.xpshuai.cn/img/image-20220415170604357.png)



调整后的R的平方越接近1，就说明模型越科学

![调整后的R的平方](http://image.xpshuai.cn/img/image-20220415170741743.png)



**5.预测模型的调优：**

![精简模型](http://image.xpshuai.cn/img/image-20220415171029466.png)



```R
anova(lm.a1)

# 如下图：说明season是对拟合误差的贡献是最小的，所以season对a1影响小，把a1去除
```

![去除a1](http://image.xpshuai.cn/img/image-20220415171523114.png)

```R
# 去除a1
lm2.a1 <- update(lm.a1, . ~. -season)
# 发现Adjusted R-squared:  0.3259 已经变小了

# 对lm.a2进行分析，发现影响最小的是Chla字段，去掉Chla字段继续更新模型，不断重复来观察R平方的值是否越变越好
lm3.a1 <- update(lm1.a, . ~. -Chla)

# 来比较两个模型
anova(lm.a1, lm2.a1)
```

![误差减小](http://image.xpshuai.cn/img/image-20220415172000961.png)

为了简化上述不断重复来减小误差，R提供了一个更加简便的函数：`step()`

```R
final.lm <- step(lm.a1) # 提升了模型预测精确度

summary(final.lm)

# 如果想要提高预测精度，也可以选用其他分析模型
```

![得到140个水样，第一种海藻的频率](http://image.xpshuai.cn/img/image-20220415172412717.png)



熟能生巧，以后多利用R来处理生活中的简单问题！

















































































































































---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/r%E8%AF%AD%E8%A8%80%E5%85%A5%E9%97%A8%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/  

