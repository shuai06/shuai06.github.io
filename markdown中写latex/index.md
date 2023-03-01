# markdown中写Latex


### 公式的两种形态
- 行内公式: $ \Large \frac{3}{12} = \frac{1}{4}$
```bash
$\Large \frac{3}{12} = \frac{1}{4}$ 
```


- 行间公式：
$$\Large \frac{3}{12} = \frac{1}{4}$$

```bash
$$\Large \frac{3}{12} = \frac{1}{4}$$

```




### 希腊字母
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220628201803.png)


### 上标、下标
> 上标与下标分别用^和_来表示。

例如`$x^2_i$`表示$x^2_i$
​
 

默认情况下，`^`和`_`只对后面的一个组产生作用，如`$10^10$`表示$10^10$
这时，我们可以用`{...}`分组。

例如`$10^{10}$`表示$10^{10}$
 

同时，`{...}`还可以消除二义性。

例如`$x^5^6$`会引发错误，而`x^{5^6}`或`{x^5}^6`则表示$x^{5^6}x$



### 数学符号
#### 运算符
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220628201837.png)

#### 关系符
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220628201859.png)


#### 省略号
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220628201915.png)



#### 求累加
sum函数：
比如用`\sum_{i=1}^{n}e^{z_i}`表示
$$\sum_{i=1}^{n}x^i$$

#### 连乘
基本连乘：`\prod  _{a}^{b}`:$\prod_{a}^{b} $

角标在上边和下边的连乘`\prod \limits_{i=1}^{n}`:$\prod \limits_{i=1}^{n}$



#### 积分，正负无穷
`$\int_a^b$ `表示: $\int_a^b$

`$\int_{- \infty}^{+ \infty}$`表示：$\int_{- \infty}^{+ \infty}$



#### 矢量
`$\vec {ab}$`表示：$\vec {ab}$ 


#### 偏导数

`$\partial x$ `表示：$\partial x$ 

`$\frac{ \partial f(x,y) }{\partial x}$`表示：$\frac{ \partial f(x,y) }{\partial x}$




#### X撇
`x{\prime}`表示$x{\prime}$





$$\pi$$


### 矩阵
暂时参考：https://blog.csdn.net/bendanban/article/details/44221279




### 括号
#### 小括号/中括号
> 使用原始的()和[]即可。

比如用`$(n+1)(n-1) = n^2-1$`表示$(n+1)(n-1) = n^2-1$

用`\left`和`\right`使括号能自适应括号内算式大小：
如：`$(\frac{x}{y})$`表示$(\frac{x}{y})$，而`$\left(\frac{x}{y}\right)$`则表示$\left(\frac{x}{y}\right)$括号大小不同



#### 大括号
> 由于Latex中大括号默认被用于分组，所以在使用大括号前，需加上转义符号\或使用\lbrace和\rbrace来表示。

比如`\{[(1+2)\times3]\times3\}+2`和`\lbrace[(1+2)\times3]\times3\rbrace+2`均表示$\lbrace[(1+2)\times3]\times3\rbrace+2$


#### 向上/下取整
- 向上取整：
分别用`\lceil`和`\rceil`表示上取整。
如：`$\lceil 3.5 \rceil$`表示$\lceil 3.5 \rceil$

- 向下取整：
分别用`\lfloor`和`\rfloor`表示下取整
如：`$\lfloor 3.5 \rfloor$`表示$\lfloor 3.5 \rfloor$






#### 绝对值
> 用`\lvert`和`\rvert`表示绝对值，或直接使用`|`

如：如`$\lvert x \rvert$`或`|x|`表示$|x|$



### 分式
#### 简单分数
> 用`$\frac ab$`表示$\frac ab$,若分子与分母不是单个字符，则用{...}分组。

如：`$\frac{a+b}{c+d}$`表示$\frac{a+b}{c+d}$


比如softmax函数的表示：


$$softmax(z_i) = \frac{e^{z_i}}{\sum_{i=1}^{n}e^{z_i}}$$





#### 连分数
> 写连分数时，用`\cfrac`代替`\frac`

如：`$$x=a_0 + \cfrac{1^2}{a_1+\cfrac{2^2}{a_2+\cfrac{3^2}{a_3+\cdots}}}$$`
表示
$$x=a_0 + \cfrac{1^2}{a_1+\cfrac{2^2}{a_2+\cfrac{3^2}{a_3+\cdots}}}$$



#### 根式
根式使用`\sqrt`表示，`$sqrt[x]{a+b}$`表示： $\sqrt[x]{a+b}$

如：`$\sqrt[3]{a^3+b^3}$`表示$\sqrt[3]{a^3+b^3}$




### 多行表达式
#### 分类表达式
> 使用`\begin{cases}...\end{cases}`，`\\`表示分类

比如：
```bash
$$
\lvert a \rvert = 
\begin{cases}
a &a>  0 \\
0 &a = 0 \\
-a &a < 0
\end{cases}
$$

```
表示：
$$
\lvert a \rvert = 
\begin{cases}
a &a>  0 \\
0 &a = 0 \\
-a &a < 0
\end{cases}
$$

若想使分类间距变大，则使用`\\[2ex]`,`\\[3ex]`…代替`\\`
```bash
$$
\lvert a \rvert =
\begin{cases}
a &a>  0 \\[2ex]
0 &a = 0 \\[2ex]
-a &a < 0
\end{cases}
$$

```

$$
\lvert a \rvert =
\begin{cases}
a &a>  0 \\[2ex]
0 &a = 0 \\[2ex]
-a &a < 0
\end{cases}
$$


#### 一行公式多行显示
> 用`\begin{split}...end{split}`表示多行公式。


如：
```bash
$$
\begin{split}
a&=b+c+d \\
&\quad +e-f \\
&=i
\end{split}
$$
```
表示：

$$
\begin{split}
a&=b+c+d \\
&\quad +e-f \\
&=i
\end{split}
$$





#### 方程组
> 用`\begin{array}...\end{array}`搭配`\left \{...\right`.表示方程组


如：
```bash
$$
\left \{
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\ 
a_2x+b_2y+c_2z=d_2 \\ 
a_3x+b_3y+c_3z=d_3
\end{array}
\right.
$$
```
表示：
$$
\left \{
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\ 
a_2x+b_2y+c_2z=d_2 \\ 
a_3x+b_3y+c_3z=d_3
\end{array}
\right.
$$



### 其它...



<script type="text/javascript" src="/js/src/bai.js"></script>





参考原文：
https://blog.csdn.net/weixin_46885185/article/details/115017109


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/markdown%E4%B8%AD%E5%86%99latex/  

