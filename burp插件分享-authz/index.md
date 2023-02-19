# Burp插件分享 Authz


<!--more-->

> 主要用来检测水平越权(读取类)和垂直越权或未授权访问漏洞


### 下载
直接在burp自带的扩展的app store下载即可：`Authz`

  


### 使用
1.拦截数据包，右键，发送到Authz    

![](https://image.geoer.cn/20230211222048.png)


可以将需要测试的多个包都发过去，等下一步批量测试  
      

2.在插件中的New Header中填写另一个账号(或者低权限账号)的cookie值(也可以是其他header值)：    

![](https://image.geoer.cn/20230211222731.png)


3.等需要收集测试的数据包差不多之后，点击`run`进行测试，如果返回的数字一样就存在未授权访问

当原响应内容长度、响应状态码和被修改后请求的响应内容长度、响应状态码一致则会`变绿`。  
也就代表着`存在越权`(仅作参考)，单击选择一行即可在下面展示出请求、响应的报文    



![](https://image.geoer.cn/20230211223046.png)

4.一个业务系统测完之后就Clear掉所有的东西  

![](https://image.geoer.cn/20230211223134.png)





### 可与HaE插件配合使用
HaE插件：对HTTP history中的数据进行正则匹配，将匹配到的数据包更改颜色  
下载地址：https://github.com/gh0stkey/HaE    



下载导入后会在用户根目录HaE文件下生成一个Config.yml文件，此文件为规则库，内容对应着该插件的匹配规则以及颜色设置，可修改Config.yml对它的规则进行添加与更改，也可以直接在burp中进行添加匹配规则      



![](https://image.geoer.cn/20230211223356.png)



**Authz插件与HaE插件一起使用：**  
在`Http History`界面，勾选只显示标注的数据包(前面自己设置的匹配规则)：    

![](https://image.geoer.cn/20230211223612.png)
`ctrl+a`全选，将所有标注的数据包选中发送到`Authz`中进行测试  

  


### 另一个插件：Authzmatrix
也是越权检测工具，但是较新  
我没有用过





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/burp%E6%8F%92%E4%BB%B6%E5%88%86%E4%BA%AB-authz/  

