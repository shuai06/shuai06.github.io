# 开源情报信息搜索(OSINT)


#### 搜索引擎语法

- 百度

- 谷歌

- 必应

  

#### 在线接口

- http://cebaidu.com/index/getRelatedSites?site_address=baidu.com
- http://www.webscan.cc/
- http://sbd.ximcx.cn/
- https://censysio/certificates?q=example.com
- https://crt.sh/?q=%25example.com
- https://github.com/cOny1WorkScripts/tree/master/get-subdomain-from-baidu
- https://dnsdumpster.com/
- https://ww.threatcrowd.org/searchApi/v2/domain/report/?domain=baidu.com
- https://findsubdomains.com/
- https://dnslytics.com/search?q=www.baidu.com
- https://pentest-tools.com/information-gathering/find-subdomains-of-domain
- https://viewdns.info/
- https://www.ipneighbour.com#/lookup/114.114.114.114
- https://securitytrails.com/list/apex_domain/baidu.com
- https://url.fht.im/
- http://api.hackertarget.com/hostsearch/?q=baidu.com
- http://ww.yunseecn/finger.html





#### 相关工具

[https://github.com/rshipp/awesome-malware-analysis/blob/main/%E6%81%B6%E6%84%8F%E8%BD%AF%E4%BB%B6%E5%88%86%E6%9E%90%E5%A4%A7%E5%90%88%E9%9B%86.md](https://github.com/rshipp/awesome-malware-analysis/blob/main/恶意软件分析大合集.md)











#### DNS历史解析记录

... ...





#### Github Hacking

可以搜索以下类型的信息：`Repositories`,`Topics`,`Issues and pull requests`,`Code`,`Commits`,`Users`,`Wikis`



###### 搜索仓库

```bash
>		exp stars:>1000	# 匹配关键字"exp"且star大于1000的仓库
>=		exp topics:>=5	# 匹配关键字"exp"且标签数量大于等于5的仓库
<		exp size:>=1000	# 匹配关键字"exp"且文件大于1KB的仓库
<=
n..*	exp stars:1000..*	# 匹配关键字"exp"且star大于等于1000的仓库
*..n	exp stars:*..1000	# 匹配关键字"exp"且star小于等于1000的仓库
n..n	exp stars:10..1000	# 匹配关键字"exp"且star大于10并小于1000的仓库

```

其他语法类似... ...



###### 搜索代码

注意事项

```bash
- 只能搜索小于384 KB的文件。
- 只能搜索少于500,000个文件的存储库。
- 登录的用户可以搜索所有公共存储库。
- 除filename 搜索外，搜索源代码时必须至少包含-一个搜索词。 例如，搜索language:javascript无效，而是这样: amazing language :javascript。
- 搜索结果最多可以显示来自同一文件的两个片段，但文件中可能会有更多结果。
- 您不能将以下通配符用作搜索查询的一部分:， ，: ; /\'"=*!?#$&+^|~<>(){}[]。搜索将忽略这些符号。

```

日期条件

```bash
exp pushed:<2020-08-08	# 搜索在2020年8月8日前push的代码,关键字是exp
exp pushed:2020-05-05..2020-08-08	# 日期区间
exp created:>=2020-08-08	# 创建时间
```

逻辑运算

```bash
AND
OR
NOT
```

排除运算

```bash
exp pushed:<2020-08-08 -language:java	# 搜索在2020年8月8日前push的代码，关键字是exp，排除java语言的仓库
```

包含搜索

```bash
exp in:file    # 搜索文件中包含exp的代码
exp in:path    # 搜索路径中包含exp的代码
exp in:file,path    # 搜索文件、路径中包含exp的代码
console path:app/public language:javascript		# 搜索关键字console,语言为js，在app/public下的代码
```

主体搜索

```bash
user:USERNAME	# 用户名搜索
org:ORGNAME		# 组织搜索
repo:USERNAME/REPOSITORY	# 指定仓库搜索
```

文件大小

```bash
size:>1000	# 搜索大于1KB的文件
```

文件名称

```bash
filename :config.php language:php 	# 搜索文件名为config.php,且语言为php的代码

# 例如搜索Java项目配置文件: 
mail filename:.properties

```

拓展名

```bash
extension:EXTENSION		# 指定扩展名搜索

# 例如:
extension:properties jdbc

```





###### 自动化工具

https://github.com/UnkL4b/GitMiner

```bash
# 简单用法
# -r 正则， -q 查询关键字
python3 gitminer-v2.0.py -C cookie.txt -q 'extension:properties jdbc' -r 'password(.*)' -m passwords

```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E5%BC%80%E6%BA%90%E6%83%85%E6%8A%A5%E4%BF%A1%E6%81%AF%E6%90%9C%E7%B4%A2-osint/  

