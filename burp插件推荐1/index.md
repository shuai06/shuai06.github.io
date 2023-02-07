# Burp插件推荐-js信息搜集插件


<!--more-->



## 简述

js文件中会包含大量前端路由和后端api的链接信息，获取到这些链接能提供更多的有效信息。





## 插件推荐

### Burp JS Link Finder(burp插件)

https://github.com/InitRoot/BurpJSLinkFinder



也可在burp的Extender下载









### burp-clj(burp插件)

burp自带的`JS Link Finder`效果很不错，不过找到的链接不能直接复制使用，需要再做处理，使用起来很不方便，**根据js-link-finder,使用burp-clj脚本重新实现**，并根据LinkFinder提供的正则表达式，新增了REST API的支持，**现在可以直接通过js link对js中找到的链接进行处理，然后作为burp Intruder的payload使用。**



**安装：**

**1.**下载burp-clj.jar，并在burp中加载该插件:https://github.com/ntestoc3/burp-clj/releases

**2.添加脚本源：**启用burp-clj插件后，切换到`Clojure Plugin`选项卡,点Add按钮添加脚本源：https://github.com/ntestoc3/burp-scripts  

(这里使用github地址，也可以git clone下来，使用本地目录)。



添加脚本源之后，点`Reload Scripts`!加载脚本。



**3.启用jslink脚本**

在Clojure Plugin选项卡的Scripts List中勾选js link parse的复选框，启用jslink脚本。
![20230207144802](http://image.xpshuai.cn/20230207144802.png)




**使用方法：**
浏览器通过burp代理访问网站，jsfind会被动扫描js文件，扫描到含有链接的js文件会生成issue。

JS Links选项卡可以看到所有扫描到的包含链接的js文件，可以同时在左侧列表选择多个js文件(按ctrl或shift多选)，右边列表会显示所有选中js文件中的链接。

针对不需要的链接可以选中后删除，然后通过 **删除链接前面的./字符**按钮统一链接的格式，方便intruder使用。
注意链接列表中的删除并不会保存，再次切换到对应的js文件，原先的链接还会存在。


**Note:**
如果浏览器访问过目标网站，则js文件会缓存，再次访问burp里面只能看到304,不会分析文件内容，需要清空缓存并硬性重新加载，用于在burp中获取js内容。

burp项目中的issue信息会保存，但jslink找到的链接不会保存，jslink脚本重新加载的话，已经分析的链接信息会丢失。







**也可以使用burp-clj来解析js map文件**：

勾选webpack map文件解析，设置保存目录

通过burp代理访问目标站点，如果webpack脚本发现存在js.map文件，就会自动下载，并解包js源码到设置的目录中．并添加相应的issue．








### jsfinder系列

> 可以自行魔改，或者自动化利用

- jsfinder:**可以批量对目标提取：**jsfinder跑一下跑出路径,然后打开burp爆破等操作

  - https://github.com/Threezh1/JSFinder
  - jsfinder的油猴插件

- jsfinderPlus:https://github.com/Roc-L8/JSFinderPlus

  









### FindSomthing(浏览器插件)

直接安装使用







### webpack敏感信息插件

Packer-Fuzzer

https://github.com/rtcatc/Packer-Fuzzer









---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/burp%E6%8F%92%E4%BB%B6%E6%8E%A8%E8%8D%901/  

