# 利用域前置Cobalt Strike逃避IDS



## 域前置(Domain Fronting)原理
> CND分发


**原理：** 通过CDN节点将流量转发到真实的C2服务器，其中CDN节点IP通过识别请求的HOST头进行流量转发，利用我们配置域名的高可信度，如我们可以设置一个微软的子域名，可以有效的躲避DLP、agent等流量检测。
域前置的核心是**CDN**

在某 cdn 服务商开通 cdn 加速服务，并将想要伪造的域名与 c2 的 ip 进行绑定（阿里云和 cloudflare 需要验证，腾讯云不需要验证域名所属），通过向 cdn 的节点 ip 发送包含想要伪造 host 的请求，cdn 便会从记录中查找该 host 对应的源站 ip，并将流量转发至原站 ip。

比如，同一个IP可以被不同的域名进行绑定加速，例如现在有两个网站分别为www.abc.com和www.zxc.com都指定同一个IP：111.111.111.111(CDN服务器)；浏览器通过HTTP请求包里的HOST头域名进行精确访问www.abc.com还是访问www.zxc.com。比如我们ping www.xxxx.com得到一个IP，通过IP反查域名,可以看到很多域名绑定在这个IP上，这个就是CDN
**CDN是通过HOST来判断你要访问那个网站的**
因为CND服务器可以达到隐藏真实IP的效果，所以C2服务器的真实IP可以得到隐藏保护。
![20220614234141](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220614234141.png)

![20220614233939](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220614233939.png)



## 域前置操作
1.登录腾讯云，开启CDN
![20220615095530](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615095530.png)


2.设置CDN的域名
![20220615095434](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615095434.png)


3.回源地址设置为VPS的地址(teamserver的真实ip)
![20220615095946](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615095946.png)

成功添加
![20220615095925](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615095925.png)


多地ping，查看IP是不同的，证明CDN生效了
![20220615110105](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615110105.png)



## 配置Cobalt Strike
**1.修改C2的profile配置文件的两处，指定Host使得CDN把请求转发到我们的服务器。**
下载C2：https://github.com/rsmudge/Malleable-C2-Profiles/
选择合适的profile文件，修改HOST头为准备好的域名（get和post都要改）,我这里选择了: https://github.com/rsmudge/Malleable-C2-Profiles/tree/master/normal  


![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615100944.png)


![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615101016.png)





**2.用c1lint检查一下**
```bash
./c2lint cdn.profile
```




**3.VPS开启teamserver**
```bash
nohup ./teamserver xxx.xx.xx.99 qweqweqwe999 cdn.profile
```


**4.cobaltstrike新建listener**

获取cdn的节点ip，超级ping这个cname获取到cdn的节点ip
如下图设置，`http hosts`设置为超级ping中的多个DNS的结点IP，`http host header`设置为申请的高可信域名
![image.png](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image.png.png)



Note:
对于下图设置我还没有经过测试不知道怎么样，有点不太理解，各种玄学问题，先去吃饭了吧:smile:
`http hosts`设置为cdn的域名，`http host header`设置为申请的高可信域名
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615101531.png)


**6.测试通过cdn是否能指向到我们的cobalt strike**
```bash
curl ca.xxx.com.dsa.dnsv1.com/wcx -H “Host: ca.xxx.com”
```



**7.生成马er，选择监听器**
运行，正常上线
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615112633.png)

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220615113147.png)


> Note: 为了方便本文采用HTTP，但实战建议使用 https

下次再弄懂一点，现在还是不是特别懂...


## 参考文章
https://mp.weixin.qq.com/s/8GBoKP3QWb0NpdllIi_YCg
https://co0ontty.github.io/2021/04/29/domain_fronting.html



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%88%A9%E7%94%A8%E5%9F%9F%E5%89%8D%E7%BD%AEcobalt-strike%E9%80%83%E9%81%BFids/  

