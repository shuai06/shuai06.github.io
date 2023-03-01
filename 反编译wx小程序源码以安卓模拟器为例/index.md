# 反编译wx小程序源码(以安卓模拟器为例)


<!--more-->

## 安卓模拟器实验
### 1.准备
1.一台ROOT的安卓手机  
2.解密工具：UnpackMiniApp.exe：https://gitee.com/steinven/wxpkg/blob/master/UnpackMiniApp.exe  
3.反编译工具：wxappUnpacker.zip：有好多人改进的，用法类似  
 我用的这个：https://github.com/HQEasy/wxappUnpacker   
https://gitee.com/steinven/wxpkg/blob/master/wxappUnpacker.zip   
https://ukm028kzyr.feishu.cn/docs/doccnW1w3vwpcnjTeTYKcdErjtK  
https://github.com/xuedingmiaojun/wxappUnpacker  
4.PC安装node环境  


> 这里先以安卓mumu模拟器为例

mumu模拟器开启root：点击右上角的三条杠进入更多选项-->进入设置中心-->基本设置-->在root权限中开启root权限-->保存关闭  

![1](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20230210180352.png)



### 2.提取微信小程序wxapkg包
> PC端和mc段也能找到wxapkg文件，但是会有加密，建议直接用手机端的  
打开微信进入测试的小程序  
然后去目录里面找：`/data/data/com.tencent.mm/MicroMsg/{User}/appbrand/pkg`  
将里面的`wxapkg`的文件导出到电脑上  
如果无法确定是哪个小程序的包，可以根据文件的修改时间来判断



**备注：**  
微信小程序文件存放路径：  

```bash
# 安卓：
/data/data/com.tencent.mm/MicroMsg/{{user哈希值}}/appbrand/pkg/

# iOS越狱：
/User/Containers/Data/Application/{{系统UUID}}/Library/WechatPrivate/{{user哈希值}}/WeApp/LocalCache/release/

# Windows：
C:\Users{{系统用户名}}\Documents\WeChat Files\Applet\

# macOS：
/Users/xps/Library/Group Containers/5A4RE8SF68.com.tencent.xinWeChat/Library/Caches/xinWeCha/{{用户hash}}/WeApp/LocalCache/release

```







### 3.解密
直接运行`UnpackMiniApp.exe`，如果是加密的包，会自动解密。  
没有加密的话，就跳过这条步骤，继续下面步骤。  

  

### 4.反编译
https://github.com/HQEasy/wxappUnpacker    

clone之后进入到`wxappUnpacker`文件夹中，安装依赖:`npm install`,  
然后执行`node wuWxapkg.js 包的路径`，即可解包到当前目录的一个以包名为名字的  新文件夹中：      

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20230210212011.png)

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20230210212342.png)

​    



**如果执行脚本中可能会出现以下报错(没报错请略过)： ** 

```
SyntaxError: Unexpected end of input
    at new Script (node:vm:100:7)
    at VMScript._compileVM (/Users/lrain/SecTools/wxappUnpacker/node_modules/vm2/lib/main.js:123:22)
    at VM.run (/Users/lrain/SecTools/wxappUnpacker/node_modules/vm2/lib/main.js:288:10)
    at /Users/lrain/SecTools/wxappUnpacker/wuWxss.js:243:27
    at /Users/lrain/SecTools/wxappUnpacker/wuLib.js:95:14
    at agent (/Users/lrain/SecTools/wxappUnpacker/wuLib.js:64:23)
    at FSReqCallback.readFileAfterClose [as oncomplete] (node:internal/fs/read_file_context:68:3)

```
解决方案：  
第一步：修改wuWxss.js文件31行    

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20230210212150.png)

```
			function statistic(data) {
            function addStat(id) {
                // if (!importCnt[id]) importCnt[id] = 1, statistic(pureData[id]);
                if(!importCnt[id]){
                  if(pureData){
                    importCnt[id]=1;
                    statistic(pureData[id]);
                  }
                }
                else ++importCnt[id];

			}

```
  第二步：修改wuWxss.js文件243行    

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20230210212237.png)

```
						// pureData = vm.run(code + "\n_C");
            pureData = vm.run(code + "}");

```


### 5.使用

**然后就是源码查看了：**  
在解包完成之后，我们可以直接审计代码  
或者也可以打开微信小程序开发者工具，选择“导入项目”，“AppID”选择测试号并导入；接着来到“本地设置”模块，  勾选上“不校验合法域名”功能,关闭JS编译成ES5 、不校验合法域名、调试基础库，然后可以愉快的开始调试对应小程序的源码了。  

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20230210204006.png)




## PC端实验
**1.打开对应小程序的目录：** 打开目录我们可以看到有很多以wx开头+16位16进制数命名的文件夹，每个文件夹下就是一个微信小程序的缓存。    
随便打开一个目录目录下名为__APP__.wxapkg包就是微信小程序的主包。  
有些小程序可能会有下面这种情况除了__APP__.wxapkg包外还有一个或多个其他.wxapkg后缀的文件，其他的文件就是也是小程序的包，可以看做是子包，对于功能比较复杂的小程序可能会有多个包。  

**2.解密：**  
PC段为了保护源码微信使用加密的方式把wxapkg包源码进行加密，加密后的文件的起始为V1MMWX。  

加密方法如下:    
```bash
首先pbkdf2生成AES的key。利用微信小程序id字符串为pass，salt为saltiest 迭代次数为1000。调用pbkdf2生成一个32位的key  
取原始的wxapkg的包得前1023个字节通过AES通过1生成的key和iv(the iv: 16 bytes),进行加密  
接着利用微信小程序id字符串的倒数第2个字符为xor key，依次异或1023字节后的所有数据，如果微信小程序id小于2位，则xorkey 为 0x66  
把AES加密后的数据（1024字节）和xor后的数据一起写入文件，并在文件头部添加V1MMWX标识  
```

  

知道解密方法可以自己写解密工具或者用现成的：UnpackMiniApp.exe或者 https://codeload.github.com/superBiuBiuMan/wechatMiniAppReverse/zip/refs/heads/main  

**3.反编译：** 跟移动端一样使用`wxappUnpacker`进行反编译  



  

  

  





## 参考文章
https://www.hackinn.com/index.php/archives/672/  
https://blog.csdn.net/hbqjzx/article/details/126215511  
https://www.cnblogs.com/lrain/p/16596405.html  
https://blog.csdn.net/weixin_43919632/article/details/124831351  



















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%8F%8D%E7%BC%96%E8%AF%91wx%E5%B0%8F%E7%A8%8B%E5%BA%8F%E6%BA%90%E7%A0%81%E4%BB%A5%E5%AE%89%E5%8D%93%E6%A8%A1%E6%8B%9F%E5%99%A8%E4%B8%BA%E4%BE%8B/  

