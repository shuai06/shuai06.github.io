# JS逆向初学笔记


<!--more-->

## JS hook技巧

通俗来讲就是js注入



**为什么hook能实现？**

因为客户端浏览器的最高的有限权限，所以在它的js运行之前给它断住，在客户端做服务端是不知道的





### hook步骤

**1.寻找要hook的点（**比如请求的header有一个参数是加密的，此时需要在header生成的时候就给他断住，请求参数就是生成的时候；如果是返回的密文，因为在前端展示需要把字符串转对象，所以可以hook住JSON.parse）

**2.编写hook逻辑**

**3.调试**



**解决了什么问题？**

相当于在浏览器注入JS

1.过各种 debugger 断点 (控制台反调试)

2.拦截请求

3.注入ws

4.主动 debugger，用户请求参数加密 (用的最多)

5.函数替换，函数置空，函数覆盖(过 debugger 有时候用)



> 参考: https://blog.csdn.net/weixin40412037/article/details/12685994







### hook方法

#### JSON.parse

JSON.parse()用于**将一个JSON 字符串转换为JS 对象**，很多网站会用到这个方法做解密相关操作

```javascript
(function() {
  var parse_ = JSON.parse;
  JSON.parse = function(jp) {
    console.log("断住了: ", jp);
    debugger;
    return parse_(jp);// 不改变原来的执行逻辑
  }})();


```

就是把JSON.parse 赋值到 parse 变量中，这时候就可以调用 parse 就相当于执行了JSON.parse，然后重写该方法，让它遇到JSON.parse 的时候就 debugger端住，然后返回原来的逻辑



这段代码更适合**在服务器发送加密信息给网站时使用**，

json.parse()的意思是将字符串json化，也就是说在hook住的时候，**前一个栈一般有解密代码，逐帧分析即可**：开始是加密的响应信息，往下调试发现变成明文，所以解密代码一般在这里面

![image-20250226100524929](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226100524929.png)

![image-20250226100542063](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226100542063.png)

就跟栈找就行了，在明文与密文之间



这里是JSON.parse，其实你想断什么方法就改成什么方法







#### JSON.stringify

JSON.stringify() 方法用于**将 JavaScript对象转换为 JSON 字符串**，在某些站点的加密过程中可能会遇到，以下代码演示了遇到 JSON.stringify() 时，则插入断点：

```javascript
(function() {
    var stringify = JSON.stringify;
    JSON.stringify = function(params) {
        console.log("Hook JSON.stringify ——> ", params);
        debugger;
        return stringify(params);
    }
})();
```









#### XHR的 URL操作



```javascript
(function () {
  var open = windows.XMLHttpRequest.prototype.open;
  windows.XMLHttpRequest.prototype.open = function (method, url, async) {
    if (url.indexOx('xxx') != -1){
      debugger;
    }
    return open.apply(this, arguments);
  };
})();
```



翻译:定义变量`open = window.XMLHttpRequest.prototype.open`

保留原始 `XMLHttpRequest.open` 方法 用于返回原来的逻辑。

然后重写`XMLHttpRequest.open` 方法，判断如果`t`参数的字符串在url就下 debugger，否则返回原来的逻辑。

- 如果是正数 表示存在里面
- 如果是-1 表示不在里面



例子：

![image-20250226100756318](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226100756318.png)

t值怎么来的？控制台注入代码：

![image-20250226100815628](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226100815628.png)









#### XMLHttpRequest 的header操作







例子：

![image-20250226100937850](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226100937850.png)



下个debuger

![image-20250226100954707](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226100954707.png)

这些是DOM和BOM对象要熟悉

**我们的精力放在找加密位置即可**







#### cookie

用的较多

eg.当cookie不为空，包含名字为v且xxx的时候

```javascript

(function () {
  var cookieTemp = '';
  Object.defineProperty(document, 'cookie', {
    set: function (val) {
      if (val.indexOf('vvvvvv') != -1) {
        debugger;
      }
      console.log('Hook捕获到cookie设置->', val);
      cookieTemp = val;
      return val;
    },
    get: function () {
      return cookieTemp;
    },
  });
})();

```





> 其他参考文章：https://mp.weixin.qq.com/s/28q2Hd0ZLRyBRyRT_8JE0g
>
> 常见hook：https://mp.weixin.qq.com/s?__biz=MjM5MDA2MTI1MA==&mid=2649142689&idx=1&sn=d63a33cba10edfb3c1a29e514a0233fa&chksm=bf114d6cabf42431437c467846969d45d512e1c994e426046d895894e3377c8b0a8e091b81b5&scene=27









## 反调试

### 常见的过debugger的方式

#### 方法1：右键“一律不在此处暂停”然后放行

![image-20250226101619563](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226101619563.png)





#### 方法2：添加条件断点

写一个false(或者写一个结果为false的等式)

然后放行

![image-20250226101656887](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226101656887.png)

注意：三个等号才是判断，比如：`1===0`









#### 方法3：停用断点

如果不需要调试页面代码，也可以简单粗暴的点击右上角图标直接禁用所有断点

![image-20250226101810856](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226101810856.png)





#### 方法4：方法置空

缺点：置空后一旦刷新页面重写的方法就无效了



##### **构造器置空**

若是鼠标右击选择一律不在此处暂停，**继续运行依旧一直在debugger，网页卡顿，且每次都生成了一个新的文件**，这种需要通过hook实现绕过，在控制台输入以下代码再回车继续执行网页就可以绕过（若是控制台报错注意当前堆栈是否在debugger处，**且以下只适用constructor的**，**若是function生成的需要更改对应的hook代码）**

如果是构造器启动的，就把构造器置空：

```javascript
var _constructor = constructor;
  Function.prototype.constructor = function(s) {
  
    if (s== "debugger") {
      console.log(s);
      return null;
    }
  return _constructor(s);
}

```

![image-20250226101926560](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226101926560.png)



##### **定时器置空**

- 第一种：`setInterval=function(){}`

在进入方法之前 ，可以下断点，在控制台把方法修改掉 `setInterval = function name(parames){}`,之后继续执行就可以绕过断点；

当然如果需要传递参数或者这个方法我们后续需要使用这个方法就不适用了



- 第二种：如果有个地方const或者let了一下定时器，我们就去油猴里面给他弄

![image-20250226102019116](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102019116.png)







#### 方法5：替换

在断点文件右击选择替换内容，选择替换文件位置，允许文件访问，注释掉debugger对应代码，ctrl+s保存，刷新网页重新运行

![image-20250226102058616](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102058616.png)

本地新建文件夹

然后允许:

![image-20250226102116336](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102116336.png)

这样就可以保存网页上下载下来的js（其实就是让他网站去加载本地另一个替换来的js）

但是这种方法只适用于有固定文件的js，但下面这种“VM”的就不行

![image-20250226102138924](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102138924.png)







#### 方法5：function关键字启动的debugger



下面这种的直接返回空

![image-20250226102208785](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102208785.png)



### F12和右键检查被禁用

#### 情况1

如下例子：

![image-20250226102241664](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102241664.png)



解决方法：打开浏览器，更多工具---开发者工具

![image-20250226102315925](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102315925.png)

然后按照前面的方式绕过反调试即可





#### 情况2

这种：“检测到非法调试”

![image-20250226102433592](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102433592.png)



解决办法：其实它就是就是检测到两个窗口------>把检查窗口独立出来就行了



然后按照前面的方式绕过反调试即可





### F12后页面空白无法显示响应内容的(about::blank)





比如：

正常打开网页可以显示内容：
![image-20250226102629164](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102629164.png)

F12抓包网站显示内容空白：

![image-20250226102650136](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102650136.png)



**解决方法：**

1.打开浏览器无痕模式，复制url在无痕浏览器中（注意此处一定不要先回车），按F12进入开发者模式，按照下图依次点击“资源source->事件监听器断点->脚本”，勾选上：

![image-20250226102720634](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102720634.png)



2.然后点击回车后会显示如下界面。此时我们将空白页的网页链接（即`about:blank`）复制到搜索框回车搜索，查看当前页面是否搜索到，若无相应内容，则点击下图数字3处逐个放行进入下一个页面，直到搜索到内容

![image-20250226102749041](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102749041.png)

![image-20250226102801502](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102801502.png)



3.经过以上操作，找到当前位置，在对应文件也就是下图数字3标识处单击右键，在出现的菜单中选择数字4标识处(Override content)替换文件

![image-20250226102848160](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102848160.png)

![image-20250226102858601](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102858601.png)





4.点击页面上方的选择文件夹，自行选择保存文件位置

![image-20250226102919216](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102919216.png)



5.之后页面上方会弹出提示，这里我们选择允许

![image-20250226102956198](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226102956198.png)

此时我们就可以正常查看具体响应内容了



另外提一下，如果新开一个界面以上操作都不对新页面起作用，或者是已经操作以上步骤后不慎关闭对应页面，**重新执行以上步骤需要清除我们之前操作的文件夹**，具体步骤如下图：

![image-20250226103017750](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226103017750.png)





**Note：**
和 `window.location="about:blank`"同理，F12进入开发者工具后出现以下情况都可以通过替换文件实现，

- window.history.back（跳转至上一级页面）
- window.close()（关闭页面）
- window.location.href = ‘https://www.baidu.com’（跳转页面到百度）



参考文章：https://blog.csdn.net/qq_43704548/article/details/143600034









> 其他参考文章：
>
> js中断点类型：https://juejin.cn/post/7041946855592165389
>
> 无限debugger绕过：https://blog.csdn.net/qq_43704548/article/details/143794106
>
> 







## nodej基础



```bash

# node执行js文件
# node js文件名
node /user/xps/1.js


# node附带一个npm（包管理器）
npm -v


#查看安装的包的列表
npm list
npm list crypto-js  //查看指定包的版本 

#npm install 模块名
npm install crypto-js --save

#有时候下载会很慢，可以更换为国内源
npm config set registry https://registry.npmmirror.com
npm config get registry  //查看当前源
npm config set registry https://registry.npmjs.org  //恢复官方源


#更新包
npm update   //默认全部更新
npm update crypto-js //更新指定包

#卸载
npm uninstall crypto-js


#cnpm  国内的安装工具 速度较快
npm install -g cnpm --registry=https://registry.npmmirror.com


```

安装完之后当前目录会多一个**node_modules**目录

和**package.json**(这个文件记录了包和对应的 拷贝时候node_modules不拷贝过去，直接执行`npm install `会读取这个文件进行包的安装)





如何使用安装的包：

```javascript
const Crypto = require('crypro-js包名')
Crypto.方法

```







## 常见算法

### base64编码



基础用法

在控制台中：

```javascript
window.btoa("123456")  //编码
'MTIzNDU2'

window.atob('MTIzNDU2')  //解码
'123456'

```





node中新版的用法：

```javascript
var a = '123456';
var b =Buffer.from(a, "utf-8").toString("base64"); //编码
console.log(b);

var c =Buffer.from(b, "base64").toString("utf-8"); //解码
console.log(c);

```

base64还能编码转图片、图片转编码





### 安装算法包

```bash
npm install crypto-js --save
```









### 哈希算法

不可逆

唯一性



用途：sign签名等







#### MD5

信息摘要算法



加密：

```javascript
const Crypto = require('crypto-js');

//加密
function md5_1(txt) {   //传递一个明文
    return Crypto.MD5(txt).toString()
}

//密文是32位
console.log(md5_1('123456'))  // e10adc3949ba59abbe56e057f20f883e

```

> 如果知道网站是md5加密的
>
> 搜索的时候搜索`Crypto`、`md5`、`md5(`、`.md5(`











#### SHA1

##### 简介

不可逆性:SHA 算法将输入数据映射为固定长度的哈希值，但无法从哈希值还原出原始数据。

唯一性:不同的输入数据很难生成相同的哈希值，保证了哈希值的唯一性.

散列性: 输入数据的微小变化会导致哈希值的显著变化，即使输入数据的长度非常大。



在实际应用中，SHA 算法常用于密码存储、数字签名、消息认证码 (MAC)和数据完整性验证等领域，以确保数据的安全性和完整性。然而，对于一些特殊需求，如密码学中的密码学哈希函数，可能需要更高级别的安全算法



##### 位数

![image-20250226103728506](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226103728506.png)



- SHA1系列
- SHA2系列：SHA256、SHA512
- SHA3系列：SHA3-512等





##### 使用

```javascript

const CryptoJS = require('crypto-js');



//加密
function sha1_encrypt(txt) {   //传递一个明文
    return CryptoJS.SHA1(txt).toString()
}

//密文是32位
console.log(sha1_encrypt('123456'))  // 7c4a8d09ca3762af61e59520943dc26494f8941b
```

















#### HMAC

##### 简介

HMAC(Hash-based Message Authentication Code)是一种基于哈希函数的消息认证码算法。它**结合了哈希函数和密钥**来提供数据完整性和身份验证的保护。 



HMAC 算法的设计目的是防止数据在传输过程中被篡改或伪造。它使用**一个密钥**和**一个哈希函数**，将密钥和要认证的数据进行处理，生成一个固定长度的认证码。接收方使用相同的密钥和哈希函数对接收到的数据进行处理，并验证生成的认证码是否与接收到的认证码匹配，从而判断数据是否完整和合法。





**HMAC 算法的主要特点包括:**

1.**密钥**:HMAC 算法使用一个密钥来增加数据的安全性。密钥只有发送方和接收方知道，用于生成和验证认证码.

2.**哈希函数**: HMAC 算法使用一个哈希函数，如SHA-256或SHA-512，来将密钥和数据进行处理。哈希函数的选择取决于所需的安全级别和性能要求。

3.数据完整性:HMAC 算法通过生成认证码来验证数据的完整性。如果数据在传输过程中被修改或篡改，接收方计算生成的认证码将与接收到的认证码不匹配，从而发现数据的算改

4.身份验证:HMAC 算法可以用于验证数据的发送方身份。由于只有发送方和接收方知道密钥，接收方可以使用相同的密钥和哈希函数对接收到的数据进行处理，并与发送方提供的认证码进行比较，从而验证发送方的身份



HMAC 算法广泛应用于网络通信、API认证、数字签名和安全协议等领域，以确保数据的完整性和身份的合法性。它提供了一种简单而有效的方法来保护数据免受篡改和伪造的攻击。





##### 使用

只需要多传一个key

HMAC的意思是加了一个key的前面的算法

```javascript
const CryptoJS = require('crypto-js');

function HMACEncrypt(){
    var text = "123456";
    var key = "xxxhukwe";  //密钥文件
    return CryptoJS. HmacMD5(text, key).toString();
    //return CryptoJS. HmacSHA1(text, key).toString();
    //return CryptoJS. HmacSHA3(text, key).toString();



}

console.log(HMACEncrypt())  //f3f2660e7627663bcc2c57effab2ccac
```











### 对称加密



#### 简介

加密密钥和解密密钥用的是同一个





#### key和iv

**数：**

- **key**：用于加密和解密数据的密钥（逆向时候这个值拿下来就行）
- **iv**：初始化向量的缩写。是在使用分组密码算法进行加密时，为了增强密码强度和安全性引入的一个参数。iv是一个固定长度的随机数或伪随机数。iv的作用是在同一个密钥加密明文时候，都能产生不同的密文，从而增加密码是随机性和安全性。



#### 算法的模式(mode)

> 有个参数需要指定来选择使用哪种模式
>
> ECB和CBC模式用的最多

1. **ECB 模式**(Electronic Codebook Mode):**将数据分成若干个固定长度的数据块，每个数据块独立加密，加密后的密文长度与原始数据块长度相同**。该模式**简单易实现，但相邻的相同数据块会得到相同的密文，容易受到重放攻击**。
2. **CBC 模式**(Cipher Block Chaining Mode): **将前一个数据块的密文与当前数据块进行异或运算后再进行加密**，可以避免 ECB 模式的问题，提高了安全性。但是该模式的加密和解密**需要依赖前一个数据块的密文，因此无法并行处理数据块**.
3. CFB 模式(Cipher Feedback Mode): 将前一个数据块的密文作为加密函数的输入，产生一个与数据块相同长度的密文，然后将该密文与当前数据块进行异或运算后再进行加密。该模式可以实现数据块的并行处理，但是需要注意密文反馈的长度问题
4. OFB 模式(Output Feedback Mode): 将前一个数据块的密文作为加密函数的输入，产生4.一个与数据块相同长度的密文，然后将该密文与当前数据块进行异或运算后再进行加密该模式可以实现数据块的并行处理，且不需要考虑密文反馈的长度问题，但是需要注意密钥流的生成方式。
5. CTR 模式(Counter Mode): 将一个计数器作为加密函数的输入，产生一个与数据块相同长度的密钥流，然后将该密钥流与当前数据块进行异或运算后再进行加密。该模式可以实现数据块的并行处理，且不需要考虑密文反馈的长度问题，但是需要注意计数器的生成方



#### padding

在使用块密码算法(如 DES、AES 等)对数据进行加密时，**如果明文数据的长度不是块长度的整数倍，那么就需要使用填充(padding)算法将明文数据填充到块长度的整数倍**。填充算法可以**保证明文数据的长度是块长度的整数倍**，**从而可以进行加密操作**。常见的填充算法包括:

1. 填充0算法:在明文数据末尾添加0，直到明文数据的长度是块长度的整数倍.
2. PKCS5和 **PKCS7** 算法: 在明文数据末尾添加填充字节，填充字节的值为需要填充的字节数，例如，如果需要填充3 个字节，那么就在末尾添加3 个字节的值为 0x03 的字节.
3. ISO 10126 算法: 在明文数据末尾添加填充字节，填充字节的值为随机生成的字节，最后个字节表示需要填充的字节数。
4. ANSI X.923算法:在明文数据末尾添加填充字节，填充字节的值为0，最后一个字节表示需要填充的字节数。

一般就是 pkcs7





#### DES

> 因为安全问题、位数问题逐渐被废弃，被AES代替



实现：

```javascript
const CryptoJS = require('crypto-js');
var key = CryptoJS.enc.Utf8.parse('das13efr')   //转一下格式
iv = CryptoJS.enc.Utf8.parse('ho8uh2e3d')
text = CryptoJS.enc.Utf8.parse('hello world')  //要加密的明文

console.log(key);        // 格式如下：{ words: [ 1684108081, 862283378 ], sigBytes: 8 }
console.log(iv);         // { words: [ 1752119413, 1748133171, 1677721600 ], sigBytes: 9 }
console.log(text);      // { words: [ 1751477356, 1864398703, 1919706112 ], sigBytes: 11 }


// 固定写法
encrypted = CryptoJS.DES.encrypt(text, key, 
    {
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
    }
    ).toString()

console.log("DES加密结果：");
console.log(encrypted);  // yaBMaxAAbpDQNlTaX8vAtQ==


//AES改一下即可
encrypted_aes = CryptoJS.AES.encrypt(text, key, 
    {
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
    }
    ).toString()

console.log("AES加密结果：");  // AES的结果要长一点
console.log(encrypted_aes);  // OszuGENj/ZgHdl4y8Gmyug==
    
```







#### AES

> 用的比较多！

##### 简介

与DES的区别

密钥长度可以为128位、192位、256位，因为位数长，加密后的值要比DES长，所以具有更高的安全性



**特点：**

1.安全性

2.灵活性：支持不同长度的密钥

3.高性能

4.**广泛应用：**用的是真的多啊



**AES同样具有key，iv，mode，padding 这些参数**





##### 使用

```javascript
const CryptoJS = require('crypto-js');
var key = CryptoJS.enc.Utf8.parse('das13efr')   //转一下格式
iv = CryptoJS.enc.Utf8.parse('ho8uh2e3d')
text = CryptoJS.enc.Utf8.parse('hello world')  //要加密的明文

console.log(key);        // 格式如下：{ words: [ 1684108081, 862283378 ], sigBytes: 8 }
console.log(iv);         // { words: [ 1752119413, 1748133171, 1677721600 ], sigBytes: 9 }
console.log(text);      // { words: [ 1751477356, 1864398703, 1919706112 ], sigBytes: 11 }


//AES改一下即可
encrypted_aes = CryptoJS.AES.encrypt(text, key, 
    {
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
    }
    ).toString()

console.log("AES加密结果：");  // AES的结果要长一点
console.log(encrypted_aes);  // OszuGENj/ZgHdl4y8Gmyug==
    
```











### 非对称加密算法



使用不同密钥进行加密和解密，有一对密钥（公钥用于加密、私钥用于解密）





**特征：**

1.一对密钥，公钥前端，私钥存于后端

2.有加密和解密

3.由于算法复杂，所以性能比对称加密慢一些，但是安全性高



> 加密后的密文每次都是不一样的，要想解密只能用私钥



#### RSA

##### 简介和使用

**1.生成密钥对：可以用**`openssl`**命令行工具**

```bash
// 生成私钥：生成2048长度的私钥，保存在rsa_private_key.pem文件中
openssl genrsa -out rsa_private_key.pem 2048


// 根据私钥生成对应的公钥
openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key_2048.pub
```

公钥：

![image-20250226104313668](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104313668.png)

私钥：

![image-20250226104331435](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104331435.png)







**2.安装一个库：jsencrypt**

```bash
npm install jsencrypt --save
```



**3.使用**

使用jsencrypt库

```javascript
window = global;
const JSEncrypt = require('jsencrypt');
var encrypt = new JSEncrypt()
const publicKey = ''
const privateKey = ''

//使用公钥加密
function encryptMessage(msg, publicKey) {
    encrypt.setPublickey(publicKey);
    var encrypted = encrypt.encrypt(msg);
    return encrypted;
}

//私钥解密
function decryptMessage(encrypted, privateKey) {
    encrypt.setPrivateKey(privateKey);
    var decrypted = encrypt.decrypt(encrypted);
    return decrypted;
}


var data = 'hello';
var encrypted = encryptMessage(data, publicKey);
console.log(encrypted);

var decrypted = encryptMessage(encrypted, privateKey);
console.log(decrypted);

```

使用node-forge库：

![image-20250226104415586](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104415586.png)



### 国密系列

https://github.com/JuneAndGreen/sm-crypto





#### SM系列



##### 简介

包括SM1，**SM2，SM3，SM4**，SM7，SM9，SUC（祖冲之加密算法）

其中：**SM2，SM3，SM4可用于商用，其他类型的涉及机密不能商用**



![image-20250226104501261](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104501261.png)

- SM2：**非对称加密**，基于椭圆曲线ECC机制的(256位)，比rsa更快(弥补了rsa慢的缺陷)(**替代rsa**)
- SM3：**哈希算法**，有对完整性的校验，安全性和效率和跟sha256差不多，压缩函数更为复杂
- SM4：**对称加密算法**，用于数据加密和局域网产品。分组分度和密钥长度均为128比特，计算轮数多(**替代对称加密算法**)



使用的库：`sm-crypto`

```bash
npm install sm-crypto  -save
```

还有个库：`gm-crypt`







##### 使用

###### SM2使用：非对称加密

```javascript
//引入库
const sm = require('sm-crypto').sm2;

//待加密的明文数据
const text = "hello";

//生成密钥对
const keypair = sm.generateKeyPairHex();

//获取公钥和私钥
const publicKey = keypair.publicKey;
const privateKey = keypair.privateKey;


//加密
const cipher_text = sm.doEncrypt(text, publicKey);
console.log("加密后：", cipher_text);  //a252f4520ac666b2b249c56c00640a88d137b2f42a6b94d1f11930cab6ec06c3d482b7aa3f94edb3fa009114c7641b0b09280116263f82e6828f442de25bdc8cc9c9fb13b97dcfde093e413c464be448350f035f84e223210a6b6ac76e71bec0cf0c8770a3

//解密
const decrypted_text = sm.doDecrypt(cipher_text, privateKey);
console.log("解密后：" ,decrypted_text);  //hello


```



###### SM3使用：哈希算法

```javascript
//引入库
const sm3 = require('sm-crypto').sm3;

var data = "11111111"

var res = sm3(data)

console.log(res);  //f5fb72062ab1d6ddc0bfecb87dc32bec6e773f585e1cbbf93de394081c2e0f37
console.log(res.length);  //64


```





###### SM4使用：对称加密

```javascript
const sm4 = require('sm-crypto').sm4
const msg = 'hello world! 我是 juneandgreen.' // 可以为 utf8 串或字节数组
const key = '0123456789abcdeffedcba9876543210' // 可以为 16 进制串或字节数组，要求为 128 比特

let encryptData = sm4.encrypt(msg, key) // 加密，默认输出 16 进制字符串，默认使用 pkcs#7 填充（传 pkcs#5 也会走 pkcs#7 填充）
//let encryptData = sm4.encrypt(msg, key, {padding: 'none'}) // 加密，不使用 padding
//let encryptData = sm4.encrypt(msg, key, {padding: 'none', output: 'array'}) // 加密，不使用 padding，输出为字节数组
//let encryptData = sm4.encrypt(msg, key, {mode: 'cbc', iv: 'fedcba98765432100123456789abcdef'}) // 加密，cbc 模式
console.log(encryptData);


let decryptData = sm4.decrypt(encryptData, key) // 解密，默认输出 utf8 字符串，默认使用 pkcs#7 填充（传 pkcs#5 也会走 pkcs#7 填充）
//let decryptData = sm4.decrypt(encryptData, key, {padding: 'none'}) // 解密，不使用 padding
//let decryptData = sm4.decrypt(encryptData, key, {padding: 'none', output: 'array'}) // 解密，不使用 padding，输出为字节数组
//let decryptData = sm4.decrypt(encryptData, key, {mode: 'cbc', iv: 'fedcba98765432100123456789abcdef'}) // 解密，cbc 模式
console.log(decryptData);




```





gm-crypt库实现：

```javascript
function SM4(config, text){
    const sm4 = require("gm-crypt").sm4;
    i = new sm4(config).encrypt(text);
    return i;
}

config = {
    "key": "1222222222fgvdsc",   //必须是16字节/字符
    "mode": "ecb",
    "cipherType":"base64"
}

console.log(SM4(config, '1234567'));  //pGpynzFE5v6GBxa560G/lw==


```











## Webpack







### 两个函数运行方法

两个执行我们调用的函数的方法：

- call
- apply

```javascript
function abc() {
  console.log(1);
}


//之前调用方法是这样的
abc()


//这样也可以调用，跟前面一样的
abc.call()    // webpack中是这样调用方法的
abc.apply()



```









### 正向打包

**1.拿到源码文件后：**

![image-20250226104727388](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104727388.png)





**2.先安装依赖包：**

```bash
npm install
```

他会自动给你装一个node_modules（里面是所有需要的模块）

![image-20250226104747254](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104747254.png)





**3.然后构建**

```bash
npm  run build
```

构建完毕之后的目录在dist中：

![image-20250226104808628](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104808628.png)

dist目录中可以看到js和css等前端文件已经按照类别存放了：

![image-20250226104823277](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104823277.png)



看一下打包完的js文件长啥样：可以看到是压缩完之后的:

![image-20250226104834099](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104834099.png)

![image-20250226104844909](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104844909.png)



4.**部署**

dist目录扔到服务器目录，做个路由就可以访问了







### 加载原理

webpack是js应用程序模块打包器，为了让js模块化方便调用

把js打包成js，只不过为了模块化调用方便，同时也缩小了体积

![image-20250226104930266](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104930266.png)



**r是加载器：**

a存放所有的模块

![image-20250226104954369](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226104954369.png)



r方法加载的是比如下面这些方法：

![image-20250226105011933](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105011933.png)



**这些**`**r.x**`**的行为就是在调加载器：前面一直用的r去调用，所以r是它的加载器，用r去调用**

![image-20250226105022260](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105022260.png)



r方法中的参数t就是下标，就是调用目录里面第几个模块：

其中a存放了所有的模块

a[t]就是加载第几个模块

![image-20250226105031259](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105031259.png)





如果a[t],说明已经缓存这个模块的导出对象了；

否则创建一个新的对象，并赋值给a[t]，并且将其缓存；

在新创建的模块i中设置了i为模块属性标识符t，

l属性如果为false则表示当前模块还没加载

.call()就是执行，是加载模块的实际操作（模块i的导出对象作为参数）

e[t]就是下面的函数

最后将其设置为true(即不等于0)

并且返回模块的导出对象



> **call()在哪里面，哪就是加载器**



### 两种加载方式





#### **调用方式1：**中括号内

![image-20250226105121565](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105121565.png)







#### 调用方式2：大括号内键值对

调用时候使用加载器调用大括号内函数前面的键即可

![image-20250226105156952](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105156952.png)



## 补充：调用堆栈

就是定义了好多代码方法，但是没有去执行

**他是通过这么去调用执行的**

**当函数被调用，就会添加到右侧这个堆栈中**

**最近被调用的在上面，以前被调用的在下面（所以从上往下找）**



![image-20250226105237024](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105237024.png)



**跟栈就是要找加密位置：**

![image-20250226105251571](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105251571.png)

![image-20250226105259302](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105259302.png)

直到找到加密位置为止。







**打印调用堆栈也可以：**

```javascript
console.trace
```

![image-20250226105321977](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250226105321977.png)











## 工具

- 爬虫工具库：https://spidertools.cn/#/
- lxtools工具库：http://cnlans.com/lx/tools
- v_jstools:https://github.com/cilame/v_jstools
- m3u8：https://github.com/Momo707577045/m3u8-downloader
- 

















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/js%E9%80%86%E5%90%91%E5%88%9D%E5%AD%A6%E7%AC%94%E8%AE%B0/  

