# JavaScript基础学习笔记


<!--more-->



## 简介

JavaScript 是一门跨平台、面向对象的脚本语言，而Java语言也是跨平台的、面向对象的语言，只不过Java是编译语言，是需要编译成字节码文件才能运行的；JavaScript是**脚本语言，不需要编译，由浏览器直接解析并执行**。JavaScript 是用来控制网页行为的，它能使网页可交互。



JavaScript（简称：JS） 在 1995 年由 Brendan Eich 发明，并于 1997 年成为一部 ECMA 标准。ECMA 规定了一套标准 就叫 `ECMAScript` ，所有的客户端校验语言必须遵守这个标准，当然 JavaScript 也遵守了这个标准。ECMAScript 6 (简称**ES6**) 是最新的 JavaScript 版本（发布于 2015 年)。



## JavaScript引入方式

JavaScript 引入方式就是 HTML 和 JavaScript 的结合方式。JavaScript引入方式有两种：

* 内部脚本：将 JS代码定义在HTML页面中
* 外部脚本：将 JS代码定义在外部 JS文件中，然后引入到 HTML页面中

### 1  内部脚本

在 HTML 中，JavaScript 代码必须位于 `<script>` 与 `</script>` 标签之间

**代码如下：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<script>
    alert("hello js");
</script>
</body>
</html>
```



>  HTML 文档中可以在任意地方，放置任意数量的<script>标签，但是一般把脚本置于 <body> 元素的底部，可改善显示速度。
>
> 因为浏览器在加载页面的时候会从上往下进行加载并解析。 我们应该让用户看到页面内容，然后再展示动态的效果。



### 2 外部脚本

**第一步：定义外部 js 文件。如定义名为 demo.js的文件**

![image-20240708144702269](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708144702269.png)

demo.js 文件内容如下：

```
alert("hello js");
```

> 外部脚本不能包含 `<script>` 标签，在js文件中直接写 js 代码即可，不要在 js文件 中写 `script` 标签
>

**第二步：在页面中引入外部的js文件**

在页面使用 `script` 标签中使用 `src` 属性指定 js 文件的 URL 路径。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<script src="../js/demo.js"></script>
</body>
</html>
```





## 基础语法

### 书写语法

* 区分大小写：与 Java 一样，变量名、函数名以及其他一切东西都是区分大小写的

* 每行结尾的分号可有可无。如果一行上写多个语句时，必须加分号用来区分多个语句。

* 注释

  * 单行注释：`// 注释内容`
  * 多行注释：`/* 注释内容 */`



- 输出语句：
  - 使用 window.alert() 写入警告框
  - 使用 document.write() 写入 HTML 输出
  - 使用 console.log() 写入浏览器控制台



### 变量

JavaScript 中用 var 关键字（variable 的缩写）来声明变量。格式 `var 变量名 = 数据值;`。而在JavaScript 是一门**弱类型语言**，变量**可以存放不同类型的值**；如下在定义变量时赋值为数字数据，还可以将变量的值改为字符串类型的数

```js
var test = 20;
test = "张三";
```

js 中的**变量名命名也有如下规则**，和java语言基本都相同

* 组成字符可以是任何字母、数字、下划线（_）或美元符号（$）
* 数字不能开头
* 建议使用驼峰命名

JavaScript 中 `var` 关键字有点特殊，有以下地方和其他语言不一样

* **作用域：全局变量**

  ```js
  {
      var age = 20;
  }
  alert(age);  // 在代码块中定义的age 变量，在代码块外边还可以使用
  ```

* **变量可以重复定义**

  ```js
  {
      var age = 20;
      var age = 30;//JavaScript 会用 30 将之前 age 变量的 20 替换掉
  }
  alert(age); //打印的结果是 30
  ```

针对如上的问题，**ECMAScript 6 新增了 `let `关键字来定义变量**。它的用法类似于 `var`，但是**所声明的变量，只在 `let` 关键字所在的代码块内有效，且不允许重复声明。**

例如：

```js
{
    let age = 20;
}
alert(age); 
```

如果定义在作用范围之外，通过 `F12` 打开开发者模式可以看到如下错误信息：

![image-20240708145451922](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708145451922.png)



如果重复定义，在IDE报错直接：

![image-20240708145333063](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708145333063.png)

同时，ECMAScript 6 新增了` const`关键字，用来声明一个**只读的常量**。一旦声明，常量的值就不能改变。

```javascript
const PI = 3.14;
PI = 3;
```



![image-20240708145531023](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708145531023.png)









### 数据类型

JavaScript 中提供了两类数据类型：原始类型 和 引用类型。

> 使用 typeof 运算符可以获取数据类型
>
> `alert(typeof age);` 以弹框的形式将 age 变量的数据类型输出

![image-20240708145813745](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708145813745.png)



原始数据类型：

* **number**：数字（整数、小数、NaN(Not a Number)）

  ```js
  var age = 20;
  var price = 99.8;
  
  alert(typeof age); // 结果是 ： number
  alert(typeof price);// 结果是 ： number
  ```

  > ==注意：== NaN是一个特殊的number类型的值，后面用到再说

* **string**：字符、字符串，单双引皆可

  ```js
  var ch = 'a';
  var name = '张三'; 
  var addr = "北京";
  
  alert(typeof ch); //结果是  string
  alert(typeof name); //结果是  string
  alert(typeof addr); //结果是  string
  ```

  > 注意：在 js 中 双引号和单引号都表示字符串类型的数据

* **boolean**：布尔。true，false

  ```js
  var flag = true;
  var flag2 = false;
  
  alert(typeof flag); //结果是 boolean
  alert(typeof flag2); //结果是 boolean
  ```

* **null**：对象为空

  ```js
  var obj = null;
  
  alert(typeof obj);//结果是 object
  ```

  为什么打印上面的 obj 变量的数据类型，结果是object；这个官方给出了解释，下面是从官方文档截的图

![image-20240708145852895](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708145852895.png)

**undefined**：当声明的变量未初始化时，该变量的默认值是 undefined

```javascript
var a ;
alert(typeof a); //结果是 undefined
```

### 运算符

JavaScript 提供了如下的运算符。大部分和Java都是一样的，不同的是 JS 关系运算符中的 `==` 和 `===`，一会我们只演示这两个的区别，其他运算符将不做演示

* 一元运算符：++，--

* 算术运算符：+，-，*，/，%

* 赋值运算符：=，+=，-=…

* 关系运算符：>，<，>=，<=，!=，\==，===…

* 逻辑运算符：&&，||，!

* 三元运算符：条件表达式 ? true_value : false_value 

#### ==和===区别

**概述:**

* ==：

  1. 判断类型是否一样，**如果不一样，则进行类型转换**

  2. 再去比较其值

* ===：js 中的全等于

  1. 判断类型是否一样，**如果不一样，直接返回false**
  2. 再去比较其值

**代码：**

```js
var age1 = 20;
var age2 = "20";

alert(age1 == age2);// true
alert(age1 === age2);// false
```

####  类型转换

* 其他类型转为number

  * string 转换为 number 类型：按照字符串的字面值，转为数字。如果字面值不是数字，则转为NaN

    将 string 转换为 number 有两种方式：

    * 使用 `+` 正号运算符：

      ```js
      var str = +"20";
      alert(str + 1) //21
      ```

    * 使用 `parseInt()` 函数(方法)：

      ```js
      var str = "20";
      alert(parseInt(str) + 1);
      ```

    > 建议使用 `parseInt()` 函数进行转换

  * boolean 转换为 number 类型：true 转为1，false转为0

    ```js
    var flag = +false;
    alert(flag); // 0
    ```

* 其他类型转为boolean

  * number 类型转换为 boolean 类型：0和NaN转为false，其他的数字转为true
  * string 类型转换为 boolean 类型：空字符串转为false，其他的字符串转为true
  * null类型转换为 boolean 类型是 false
  * undefined 转换为 boolean 类型是 false

  **代码如下：**

  ```js
  // var flag = 3;
  // var flag = "";
  var flag = undefined;
  
  if(flag){
      alert("转为true");
  }else {
      alert("转为false");
  }
  ```

**使用场景：**

在 Java 中使用字符串前，一般都会先判断字符串不是null，并且不是空字符才会做其他的一些操作，JavaScript也有类型的操作，代码如下：

```js
var str = "abc";

//健壮性判断
if(str != null && str.length > 0){
    alert("转为true");
}else {
    alert("转为false");
}
```

但是由于 JavaScript 会自动进行类型转换，所以上述的判断可以进行简化，代码如下：

```js
var str = "abc";

//健壮性判断
if(str){
    alert("转为true");
}else {
    alert("转为false");
}
```











### 流程控制语句

JavaScript 中提供了和 Java 一样的流程控制语句，如下

* if 
* switch
* for
* while
* dowhile







####  if 语句

```js
var count = 3;
if (count == 3) {
    alert(count);
}
```

#### switch 语句

```js
var num = 3;
switch (num) {
    case 1:
        alert("星期一");
        break;
    case 2:
        alert("星期二");
        break;
    case 3:
        alert("星期三");
        break;
    case 4:
        alert("星期四");
        break;
    case 5:
        alert("星期五");
        break;
    case 6:
        alert("星期六");
        break;
    case 7:
        alert("星期日");
        break;
    default:
        alert("输入的星期有误");
        break;
}
```

#### for 循环语句

```js
var sum = 0;
for (let i = 1; i <= 100; i++) { //建议for循环小括号中定义的变量使用let
    sum += i;
}
alert(sum);
```

#### while 循环语句

```js
var sum = 0;
var i = 1;
while (i <= 100) {
    sum += i;
    i++;
}
alert(sum);
```

#### do while 循环语句

```js
var sum = 0;
var i = 1;
do {
    sum += i;
    i++;
}
while (i <= 100);
alert(sum);
```

### 函数

函数（就是Java中的方法）是被设计为执行特定任务的代码块；JavaScript 函数通过 function 关键词进行定义。

####  定义格式

函数定义格式有两种：

* 方式1

  ```js
  function 函数名(参数1,参数2..){
      要执行的代码
  }
  ```

* 方式2

  ```js
  var 函数名 = function (参数列表){
      要执行的代码
  }
  ```

> 注意：
>
> * 形式参数不需要类型。因为JavaScript是弱类型语言
>
>   ```js
>   function add(a, b){
>       return a + b;
>   }
>   ```
>
>   上述函数的参数 a 和 b 不需要定义数据类型，因为在每个参数前加上 var 也没有任何意义。
>
> * 返回值也不需要定义类型，可以在函数内部直接使用return返回即可

#### 函数调用

函数调用函数：

```js
函数名称(实际参数列表);
```

例如：

```js
let result = add(10,20);
```

> 注意：
>
> * JS中，函数调用可以传递**任意个数**参数，例如  `let result = add(1,2,3);` 
>
>   它是将数据 1 传递给了变量a，将数据 2 传递给了变量 b，而数据 3 没有变量接收。

## JavaScript常用对象

总共分类三类：

- 基本对象
  - ![image-20240708150359804](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708150359804.png)
- BOM对象
  - ![image-20240708150407709](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708150407709.png)
- DOM对象
  - 比较多，部分示例如下：
  - ![image-20240708150435160](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708150435160.png)





### 基本对象之Array

JavaScript Array对象用于定义数组



#### 定义格式

数组的定义格式有两种：

* 方式1

  ```js
  var 变量名 = new Array(元素列表); 
  ```

  例如：

  ```js
  var arr = new Array(1,2,3); //1,2,3 是存储在数组中的数据（元素）
  ```

* 方式2

  ```js
  var 变量名 = [元素列表];
  ```

  例如：

  ```js
  var arr = [1,2,3]; //1,2,3 是存储在数组中的数据（元素）
  ```

  > 注意：Java中的数组静态初始化使用的是{}定义，而 JavaScript 中使用的是 [] 定义

#### 元素访问

访问数组中的元素和 Java 语言的一样，格式如下：

```js
arr[索引] = 值;
```

**代码演示：**

```js
 // 方式一
var arr = new Array(1,2,3);
// alert(arr);

// 方式二
var arr2 = [1,2,3];
//alert(arr2);

// 访问
arr2[0] = 10;
alert(arr2)
```

#### 特点

JavaScript 中的数组相当于 Java 中集合。**数组的长度是可以变化的**，而 JavaScript 是弱类型，所以可以存储任意的类型的数据。

例如如下代码：

```js
// 变长
var arr3 = [1,2,3];
arr3[10] = 10;
alert(arr3[10]); // 10
alert(arr3[9]);  //undefined
```

上面代码在定义数组中给了三个元素，又给索引是 10 的位置添加了数据 10，那么 `索引3` 到 `索引9` 位置的元素是什么呢？我们之前就介绍了，在 JavaScript 中没有赋值的话，默认就是 `undefined`。

如果给 `arr3` 数组添加字符串的数据，也是可以添加成功的

```js
arr3[5] = "hello";
alert(arr3[5]); // hello
```

#### 属性

Array 对象提供了很多属性，这里以`length`为例：

```javascript
var arr = [1,2,3];
for (let i = 0; i < arr.length; i++) {   //可以动态的获取数组的长度，可以遍历数组
    alert(arr[i]);
}
```





#### 方法

Array 对象同样也提供了很多方法

![image-20240708150903684](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708150903684.png)



以 `push` 函数和 `splice` 函数为例：

* push 函数：给数组添加元素，也就是在数组的末尾添加元素

  参数表示要添加的元素

  ```js
  // push:添加方法
  var arr5 = [1,2,3];
  arr5.push(10);
  alert(arr5);  //数组的元素是 {1,2,3,10}
  ```

* splice 函数：删除元素

  参数1：索引。表示从哪个索引位置删除

  参数2：个数。表示删除几个元素

  ```js
  // splice:删除元素
  var arr5 = [1,2,3];
  arr5.splice(0,1); //从 0 索引位置开始删除，删除一个元素 
  alert(arr5); // {2,3}
  ```

### String

String对象的创建方式有两种

* 方式1：

  ```js
  var 变量名 = new String(s); 
  ```

* 方式2：

  ```js
  var 变量名 = "数组"; 
  ```

**属性：**

String对象提供了很多属性，以 `length`为例：该属性是用于动态的获取字符串的长度![image-20240708151035641](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708151035641.png)



**函数：**

同样有很多，以`charAt()`返回在指定位置的字符，和`indexOf()`检索字符串，和`trim()`去掉字符串两端字符串(比如登录时候去掉用户输入的空格)为例：

```
var s = 'abcd';
console.log(s.charAt(1));
console.log(s.indexOf('c'));
```

![image-20240708151355977](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708151355977.png)



```
var str4 = '  abc   ';
alert(1 + str4 + 1);
```

去掉空格：

```
var str4 = '  abc   ';
alert(1 + str4.trim() + 1);
```













### 自定义对象

自定义对象的格式：

```javascript
var 对象名称 = {
    属性名称1:属性值1,
    属性名称2:属性值2,
    ...,
    函数名称:function (形参列表){},
	...
};
```



调用属性的格式：

```js
对象名.属性名
```

调用函数的格式：

```js
对象名.函数名()
```

例子：

```javascript
var person = {
        name : "zhangsan",
        age : 23,
        eat: function (){
            alert("干饭~");
        }
    };


alert(person.name);  //zhangsan
alert(person.age); //23

person.eat();  //干饭~
```

> 有那么一点点点像Java中的类一样





## BOM

BOM：Browser Object Model **浏览器对象模型**。也就是 JavaScript 将浏览器的各个组成部分封装为对象。



我们要操作浏览器的各个组成部分就可以通过操作 BOM 中的对象来实现。比如：我现在想将浏览器地址栏的地址改为 `https://www.itheima.com` 就可以通过使用 BOM 中定义的 `Location` 对象的 `href` 属性，代码： `location.href = "https://itheima.com";` 



 BOM 中包含了如下对象：

* Window：浏览器窗口对象
* Navigator：浏览器对象
* Screen：屏幕对象
* History：历史记录对象
* Location：地址栏对象



下图是 BOM 中的各个对象和浏览器的各个组成部分的对应关系：

![image-20240708151804401](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708151804401.png)



### Window对象

window 对象是 JavaScript 对浏览器的窗口进行封装的对象。

#### 获取window对象:

该对象不需要创建直接使用 `window`，其中 `window. ` 可以省略。比如我们之前使用的 `alert()` 函数，其实就是 `window` 对象的函数，在调用是可以写成如下两种

* 显式使用 `window` 对象调用

  ```js
  window.alert("abc");
  ```

* 隐式调用

  ```
  alert("abc")
  ```

  


 #### window对象属性

`window` 对象提供了用于获取其他 BOM 组成对象的属性

![image-20240708151948647](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708151948647.png)

也就是说，我们想使用 `Location` 对象的话，就可以使用 `window` 对象获取；写成 `window.location`，而 `window.` 可以省略，简化写成 `location` 来获取 `Location` 对象。

#### window对象函数

`window` 对象提供了很多函数供我们使用，常用如下：

![image-20240708152020174](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708152020174.png)



**定时器代码演示：**

```js
setTimeout(function (){
    alert("hehe");
},3000);
```

当我们打开浏览器，3秒后才会弹框输出 `hehe`，并且只会弹出一次。



```js
setInterval(function (){
    alert("hehe");
},2000);
```

当我们打开浏览器，**每隔2秒**都会弹框输出 `hehe`。



### History对象

History 对象是 JavaScript 对历史记录进行封装的对象。

* History 对象的获取

  使用 window.history获取，其中window. 可以省略

* History 对象的函数

![image-20240708152335091](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708152335091.png)





### Location对象

Location 对象是 JavaScript 对地址栏封装的对象。可以通过操作该对象，跳转到任意页面。



#### 获取Location对象

使用 window.location获取，其中window. 可以省略

```js
window.location.方法();
location.方法();
```

#### Location对象属性

Location对象提供了很对属性。以后常用的只有一个属性 `href`

```
alert("要跳转了");
location.href = "https://www.baidu.com";
```





### DOM

DOM：Document Object Model **文档对象模型**。也就是 JavaScript **将 HTML 文档的各个组成部分封装为对象。**



类似XML，XML 文档中的标签需要我们写代码解析，而 HTML 文档是浏览器解析。封装的对象分为

* Document：整个文档对象
* Element：元素对象
* Attribute：属性对象
* Text：文本对象
* Comment：注释对象

如下图，左边是 HTML 文档内容，右边是 DOM 树

![image-20240708152603292](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708152603292.png)

**作用：**

**JavaScript 通过 DOM， 就能够对 HTML进行操作了**

* 改变 HTML 元素的内容
* 改变 HTML 元素的样式（CSS）
* 对 HTML DOM 事件作出反应
* 添加和删除 HTML 元素

**DOM相关概念：**

DOM 是 W3C（万维网联盟）定义了访问 HTML 和 XML 文档的标准。该标准被分为 3 个不同的部分：

1. 核心 DOM：针对任何结构化文档的标准模型。 XML 和 HTML 通用的标准

   * Document：整个文档对象

   * Element：元素对象

   * Attribute：属性对象

   * Text：文本对象

   * Comment：注释对象

2. XML DOM： 针对 XML 文档的标准模型

3. HTML DOM： 针对 HTML 文档的标准模型

   该标准是在核心 DOM 基础上，对 HTML 中的每个标签都封装成了不同的对象

   * 例如：`<img>` 标签在浏览器加载到内存中时会被封装成 `Image` 对象，同时该对象也是 `Element` 对象。
   * 例如：`<input type='button'>` 标签在浏览器加载到内存中时会被封装成 `Button` 对象，同时该对象也是 `Element` 对象



###  获取 Element对象

HTML 中的 Element 对象可以通过 `Document` 对象获取，而 `Document` 对象是通过 `window` 对象获取。

`Document` 对象中提供了以下获取 `Element` 元素对象的函数

* `getElementById()`：根据id属性值获取，返回单个Element对象
* `getElementsByTagName()`：根据标签名称获取，返回Element对象数组
* `getElementsByName()`：根据name属性值获取，返回Element对象数组
* `getElementsByClassName()`：根据class属性值获取，返回Element对象数组



html测试页面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <img id="light" src="../1.png"> <br>

    <div class="cls">测试</div>   <br>
    <div class="cls">hh</div> <br>

    <input type="checkbox" name="hobby"> 电影
    <input type="checkbox" name="hobby"> 旅游
    <input type="checkbox" name="hobby"> 游戏
    <br>
    <script>
		//在此处书写js代码
    </script>
</body>
</html>
```

**1.根据 `id` 属性值获取上面的 `img` 元素对象，返回单个对象**

```js
var img = document.getElementById("light");
alert(img);
```

结果如下：

![image-20240708152902955](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708152902955.png)



**2.根据标签名称获取所有的 `div` 元素对象**

```js
var divs = document.getElementsByTagName("div");// 返回一个数组，数组中存储的是 div 元素对象
// alert(divs.length);  //输出 数组的长度
//遍历数组
for (let i = 0; i < divs.length; i++) {
    alert(divs[i]);
}
```

**3.获取所有的满足 `name = 'hobby'` 条件的元素对象**

```js
//3. getElementsByName：根据name属性值获取，返回Element对象数组
var hobbys = document.getElementsByName("hobby");
for (let i = 0; i < hobbys.length; i++) {
    alert(hobbys[i]);
}
```

**4.获取所有的满足 `class='cls'` 条件的元素对象**

```js
//4. getElementsByClassName：根据class属性值获取，返回Element对象数组
var clss = document.getElementsByClassName("cls");
for (let i = 0; i < clss.length; i++) {
    alert(clss[i]);
}
```

### HTML Element对象使用



比如前面html代码中将div标签中内容替换为`哈哈`:

```javascript
//1，获取所有的 div 元素对象
var divs = document.getElementsByTagName("div");
/*
        style:设置元素css样式
        innerHTML：设置元素内容
    */
//2，遍历数组，获取到每一个 div 元素对象，并修改元素内容
for (let i = 0; i < divs.length; i++) {
    //divs[i].style.color = 'red';
    divs[i].innerHTML = "呵呵";
}
```



让所有复选框变为勾选状态：

```javascript
//1，获取所有的 复选框 元素对象
var hobbys = document.getElementsByName("hobby");
//2，遍历数组，通过将 复选框 元素对象的 checked 属性值设置为 true 来改变复选框的选中状态
for (let i = 0; i < hobbys.length; i++) {
    hobbys[i].checked = true;
}
```







## 事件

HTML 事件是发生在 HTML 元素上的“事情”。比如：页面上的 `按钮被点击`、`鼠标移动到元素之上`、`按下键盘按键` 等都是事件。

**事件监听**是JavaScript 可以**在事件被侦测到时执行一段逻辑代码**。



比如下图输入框，当我们输入了用户名 `光标离开` 输入框，就需要通过 js 代码对输入的内容进行校验，没通过校验就在输入框后提示 `用户名格式有误!`

![image-20240708153226101](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240708153226101.png)



### 事件绑定

JavaScript 提供了两种事件绑定方式：

* 方式一：通过 HTML标签中的事件属性进行绑定

  如下面代码，有一个按钮元素，我们是在该标签上定义 `事件属性`，在事件属性中绑定函数。`onclick` 就是 `单击事件` 的事件属性。`onclick='on（）'` 表示该点击事件绑定了一个名为 `on()` 的函数

  ```html
  <input type="button" onclick='on()’>
  ```

  下面是点击事件绑定的 `on()` 函数

  ```js
  function on(){
  	alert("我被点了");
  }
  ```

* 方式二：通过 DOM 元素属性绑定

  如下面代码是按钮标签，在该标签上我们并没有使用 `事件属性`，绑定事件的操作需要在 js 代码中实现

  ```html
  <input type="button" id="btn">
  ```

  下面 js 代码是获取了 `id='btn'` 的元素对象，然后将 `onclick` 作为该对象的属性，并且绑定匿名函数。该函数是在事件触发后自动执行

  ```js
  document.getElementById("btn").onclick = function (){
      alert("我被点了");
  }
  ```

**代码演示：**

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <!--方式1：在下面input标签上添加 onclick 属性，并绑定 on() 函数-->
    <input type="button" value="点我" onclick="on()"> <br>
    <input type="button" value="再点我" id="btn">

    <script>
        function on(){
            alert("我被点了");
        }
      	//方式2：获取 id="btn" 元素对象，通过调用 onclick 属性 绑定点击事件
        document.getElementById("btn").onclick = function (){
            alert("我被点了");
        }
    </script>
</body>
</html>
```



### 常见事件

| 事件属性名  | 说明                     |
| ----------- | ------------------------ |
| onclick     | 鼠标单击事件             |
| onblur      | 元素失去焦点             |
| onfocus     | 元素获得焦点             |
| onload      | 某个页面或图像被完成加载 |
| onsubmit    | 当表单提交时触发该事件   |
| onmouseover | 鼠标被移到某元素之上     |
| onmouseout  | 鼠标从某元素移开         |



比如`onsubmit`表单提交事件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form id="register" action="#" >
        <input type="text" name="username" />
        <input type="submit" value="提交">
    </form>
    <script>
        
    </script>
</body>
</html>
```

当我们点击 `提交` 按钮后，表单就会提交，此处默认使用的是 `GET` 提交方式，会将提交的数据拼接到 URL 后。现需要通过 js 代码实现阻止表单提交的功能，js 代码实现如下：

1. 获取 `form` 表单元素对象。
2. 给 `form` 表单元素对象绑定 `onsubmit` 事件，并绑定匿名函数。
3. 该匿名函数如果返回的是true，提交表单；如果返回的是false，阻止表单提交。

```js
document.getElementById("register").onsubmit = function (){
    //onsubmit 返回true，则表单会被提交，返回false，则表单不提交
    return true;
}
```



### 案例

注册页面，对表单进行校验，如果输入的用户名、密码、手机号符合规则，则允许提交；如果不符合规则，则不允许提交。

完成以下需求：

1. 当输入框失去焦点时，验证输入内容是否符合要求

2. 当点击注册按钮时，判断所有输入框的内容是否都符合要求，如果不合符则阻止表单提交



初始界面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>欢迎注册</title>
    <link href="../css/register.css" rel="stylesheet">
</head>
<body>
    <div class="form-div">
        <div class="reg-content">
            <h1>欢迎注册</h1>
            <span>已有帐号？</span> <a href="#">登录</a>
        </div>
        <form id="reg-form" action="#" method="get">
            <table>
                <tr>
                    <td>用户名</td>
                    <td class="inputs">
                        <input name="username" type="text" id="username">
                        <br>
                        <span id="username_err" class="err_msg" style="display: none">用户名不太受欢迎</span>
                    </td>
                </tr>

                <tr>
                    <td>密码</td>
                    <td class="inputs">
                        <input name="password" type="password" id="password">
                        <br>
                        <span id="password_err" class="err_msg" style="display: none">密码格式有误</span>
                    </td>
                </tr>

                <tr>
                    <td>手机号</td>
                    <td class="inputs"><input name="tel" type="text" id="tel">
                        <br>
                        <span id="tel_err" class="err_msg" style="display: none">手机号格式有误</span>
                    </td>
                </tr>
            </table>
            <div class="buttons">
                <input value="注 册" type="submit" id="reg_btn">
            </div>
            <br class="clear">
        </form>

    </div>


    <script>

    </script>
</body>
</html>
```



**输入框验证规则：**

* 校验用户名。当用户名输入框失去焦点时，判断输入的内容是否符合 `长度是 6-12 位` 规则，不符合使 `id='username_err'` 的span标签显示出来，给出用户提示。
* 校验密码。当密码输入框失去焦点时，判断输入的内容是否符合 `长度是 6-12 位` 规则，不符合使 `id='password_err'` 的span标签显示出来，给出用户提示。
* 校验手机号。当手机号输入框失去焦点时，判断输入的内容是否符合 `长度是 11 位` 规则，不符合使 `id='tel_err'` 的span标签显示出来，给出用户提示。

```javascript
//1. 验证用户名是否符合规则
//1.1 获取用户名的输入框
var usernameInput = document.getElementById("username");

//1.2 绑定onblur事件 失去焦点
usernameInput.onblur = function () {
    //1.3 获取用户输入的用户名
    var username = usernameInput.value.trim();

    //1.4 判断用户名是否符合规则：长度 6~12
    if (username.length >= 6 && username.length <= 12) {
        //符合规则
        document.getElementById("username_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("username_err").style.display = '';
    }
}

//1. 验证密码是否符合规则
//1.1 获取密码的输入框
var passwordInput = document.getElementById("password");

//1.2 绑定onblur事件 失去焦点
passwordInput.onblur = function() {
    //1.3 获取用户输入的密码
    var password = passwordInput.value.trim();

    //1.4 判断密码是否符合规则：长度 6~12
    if (password.length >= 6 && password.length <= 12) {
        //符合规则
        document.getElementById("password_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("password_err").style.display = '';
    }
}

//1. 验证手机号是否符合规则
//1.1 获取手机号的输入框
var telInput = document.getElementById("tel");

//1.2 绑定onblur事件 失去焦点
telInput.onblur = function() {
    //1.3 获取用户输入的手机号
    var tel = telInput.value.trim();

    //1.4 判断手机号是否符合规则：长度 11
    if (tel.length == 11) {
        //符合规则
        document.getElementById("tel_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("tel_err").style.display = '';
    }
}
```



**表单验证要求：**

当用户点击 `注册` 按钮时，需要同时对输入的 `用户名`、`密码`、`手机号` ，如果都符合规则，则提交表单；如果有一个不符合规则，则不允许提交表单。实现该功能需要获取表单元素对象，并绑定 `onsubmit` 事件

```javascript
//1. 获取表单对象
var regForm = document.getElementById("reg-form");

//2. 绑定onsubmit 事件
regForm.onsubmit = function () {
    
}
```

```javascript
//`onsubmit` 事件绑定的函数需要对输入的 `用户名`、`密码`、`手机号` 进行校验，这些校验我们之前都已经实现过了，这里我们还需要再校验一次吗？不需要，只需要对之前校验的代码进行改造，把每个校验的代码专门抽象到有名字的函数中，方便调用；并且每个函数都要返回结果来去决定是提交表单还是阻止表单提交，代码如下：

//1. 验证用户名是否符合规则
//1.1 获取用户名的输入框
var usernameInput = document.getElementById("username");

//1.2 绑定onblur事件 失去焦点
usernameInput.onblur = checkUsername;

function checkUsername() {
    //1.3 获取用户输入的用户名
    var username = usernameInput.value.trim();

    //1.4 判断用户名是否符合规则：长度 6~12
    var flag = username.length >= 6 && username.length <= 12;
    if (flag) {
        //符合规则
        document.getElementById("username_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("username_err").style.display = '';
    }
    return flag;
}

//1. 验证密码是否符合规则
//1.1 获取密码的输入框
var passwordInput = document.getElementById("password");

//1.2 绑定onblur事件 失去焦点
passwordInput.onblur = checkPassword;

function checkPassword() {
    //1.3 获取用户输入的密码
    var password = passwordInput.value.trim();

    //1.4 判断密码是否符合规则：长度 6~12
    var flag = password.length >= 6 && password.length <= 12;
    if (flag) {
        //符合规则
        document.getElementById("password_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("password_err").style.display = '';
    }
    return flag;
}

//1. 验证手机号是否符合规则
//1.1 获取手机号的输入框
var telInput = document.getElementById("tel");

//1.2 绑定onblur事件 失去焦点
telInput.onblur = checkTel;

function checkTel() {
    //1.3 获取用户输入的手机号
    var tel = telInput.value.trim();

    //1.4 判断手机号是否符合规则：长度 11
    var flag = tel.length == 11;
    if (flag) {
        //符合规则
        document.getElementById("tel_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("tel_err").style.display = '';
    }
    return flag;
}


```

而 `onsubmit` 绑定的函数需要调用 `checkUsername()` 函数、`checkPassword()` 函数、`checkTel()` 函数。

```javascript
//1. 获取表单对象
var regForm = document.getElementById("reg-form");

//2. 绑定onsubmit 事件
regForm.onsubmit = function () {
    //挨个判断每一个表单项是否都符合要求，如果有一个不合符，则返回false

    var flag = checkUsername() && checkPassword() && checkTel();

    return flag;
}
```









## 正则表达式

### 正则对象

在 js 中对正则表达式封装的对象就是正则对象。

#### 创建对象

正则对象有两种创建方式：

* 直接量方式：注意不要加引号

  ```js
  var reg = /正则表达式/;
  ```

* 创建 RegExp 对象

  ```js
  var reg = new RegExp("正则表达式");
  ```

#### 函数

`test(str)` ：判断指定字符串是否符合规则，返回 true或 false





### 正则表达式

正则表达式**定义了字符串组成的规则**。也就是判断指定的字符串是否符合指定的规则，如果符合返回true，如果不符合返回false。

正则表达式是**和语言无关**的。很多语言都支持正则表达式，Java语言也支持，只不过正则表达式在不同的语言中的使用方式不同，js 中需要使用正则对象来使用正则表达式。



正则表达式**常用的规则如下**：

* ^：表示开始

* $：表示结束

* [ ]：代表某个范围内的单个字符，比如： [0-9] 单个数字字符

* .：代表任意单个字符，除了换行和行结束符

* \w：代表单词字符：字母、数字、下划线(_)，相当于 [A-Za-z0-9_]

* \d：代表数字字符： 相当于 [0-9]

量词：

* +：至少一个

* *：零个或多个

* ？：零个或一个

* {x}：x个

* {m,}：至少m个

* {m,n}：至少m个，最多n个



例子：

```javascript
// 规则：单词字符，6~12
//1,创建正则对象，对正则表达式进行封装
var reg = /^\w{6,12}$/;

var str = "abcccc";
//2,判断 str 字符串是否符合 reg 封装的正则表达式的规则
var flag = reg.test(str);
alert(flag);
```



这样，前面验证注册表单的例子中：

```javascript
//判断用户名是否符合规则：长度 6~12,单词字符组成
var reg = /^\w{6,12}$/;
var flag = reg.test(username);



//判断手机号是否符合规则：长度 11，数字组成，第一位是1
var reg = /^[1]\d{10}$/;
var flag = reg.test(tel);
if (flag) {
  //符合规则
  document.getElementById("tel_err").style.display = 'none';
} else {
  //不合符规则
  document.getElementById("tel_err").style.display = '';
```





































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/javascript%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/  

