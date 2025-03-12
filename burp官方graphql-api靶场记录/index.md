# Burp官方GraphQL API靶场记录


<!--more-->





### GraphQL API简介

#### 简介

GraphQL 是一个用于API的查询语言，使用基于类型系统来执行查询的服务（类型系统由你的数据定义）。**GraphQL并没有和任何特定数据库或者存储引擎绑定**，而是依靠你现有的代码和数据支撑。GraphQL 的作用类似：**前端可以按需“点”数据，后端精准返回**



在REST API中，往往我们的请求需要多个API，每个API是一个类型。

比如：`http://www.test.com/users/{id}` 这个API可以获取用户的信息，`http://www.test.com/users/list` 这个API可以获取所有用户的信息

**在graphql中**则不需要这么多api来实现不同的功能，**只需要一个API**，比如：`http://www.test.com/graphql`这个API。

查询不同的内容仅需要改变post内容，不再需要维护多个api。

打开GraphQL的官方demo尝试一下：https://graphql.org/swapi-graphql

比如查询`personID`为1的人的`id`和`birthYear`信息：

```json
query{
  person(personID: 1){
    id		//这里是要拿什么字段，这里要拿id和birthYear返回给我们
    birthYear
}
}
```



![image-20250311145214184](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311145214184.png)



在这种架构中，客户端首先与GraphQL进行交互，后者又与后端逻辑代码进行交互，最终逻辑代码与数据库进行信息交互。描述这种情况的图是：

![image-20250311145303343](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311145303343.png)



这种架构的优势如下：

1. 可以在单个请求中获得客户端所需的所有数据（而REST API需要执行多个请求）
2. 使用一个Endpoint（URL路由）即可处理多种请求。





最常见的查询方式有三种：**query（查询）、mutation（变更）、subscription（订阅）**。



- query（查询）：用于获取数据，只读取不修改：

  ```json
  query 操作名称(可选参数) {
    字段名(参数) {
      子字段
    }
  }
  ```

- Mutation（变更）:用于修改数据，属于写操作，会改变服务器状态：

  ```json
  mutation 操作名称(输入参数) {
    操作名(输入) {
      返回的字段
    }
  }
  ```



- Subscription（订阅）：实时监听数据变化（类似 WebSocket），属于长连接，服务器主动推送数据：

```json
subscription 操作名称 {
  监听的事件名 {
    返回的字段
  }
}
```



**内省（Introspection）：** 是一个内置的 GraphQL 函数，可让您查询服务器以获取有关架构的信息

自省可能带来严重的信息泄露风险，因为它可能被用来访问潜在的敏感信息（例如字段描述）并帮助攻击者了解如何与 API 交互。在生产环境中禁用自省是最佳做法







#### 渗透测试

##### 识别

**1.接口路径比较固定，常见的api路径如下：**

```
/graphql
/graphiql
/v1/graphql
/v2/graphql
/v3/graphql
/v1/graphiql
/v2/graphiql
/v3/graphiql
/api/graphql
/api/graphiql
/graphql/api
/graphql/console
/console
/playground
/gql
/query
/graphql-devtools
/graphql-explorer
/graphql-playground
/graphql.php
/index.php?graphql
/graphql
/graphql-console
/graphql-devtools
/graphql-explorer
/graphql-playground
/graphql-playground-html
/graphql.php
/graphql/console
/graphql/graphql
/graphql/graphql-playground
/graphql/schema.json
/graphql/schema.xml
/graphql/schema.yaml
/graphql/v1
/HyperGraphQL
/je/graphql
/laravel-graphql-playground
/lol/graphql
/portal-graphql
/v1/api/graphql
/v1/graphql
/v1/graphql-explorer
/v1/graphql.php
/v1/graphql/console
/v1/graphql/schema.json
/v1/graphql/schema.xml
/v1/graphql/schema.yaml
/v2/api/graphql
/v2/graphql
/v2/graphql-explorer
/v2/graphql.php
/graph
/graphql/console/
/graphiql
/graphiql.php
/api
............
```





**2.此外，一些不正确的请求（POST->GET）或者参数缺失可能导致报错，可借此判断：**





**3.通过一些比较通用的payload进行请求尝试，比如`query{__typename}`**

例如，在fofa找一个graphql报错的页面：`body="Must provide query string" || body="GraphQL validation failed"`

然后使用POST方法传一个通用的payload(`{"query":"query{__typename}"}`)进行请求探测

> 注意`Content-Type`最好改成`application/json`

可以看到响应变成200，查询成功，如此即可判断网站使用了graphql API，请求成功后burp会自动在请求这里新增一个GraphQL模块，可以直接在这里构造graphql请求：

![image-20250311204837088](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311204837088.png)

比如使用`__schema`字段进行**内省查询**，这个字段在所有查询的根类型上都可用：

查询接口中的可用类型：

```json
query {
  __schema {
    types {
      name
    }
  }
}
```





##### 攻击面

> burp插件`InQL`

在测试 GraphQL API 之前，首先需要**找到其端点**（由于 GraphQL API 对所有请求都使用相同的端点，因此这是一条很有价值的信息）

- 发送`query{__typename}`到任何 GraphQL 端点，它将`{"data": {"__typename": "query"}}`在其响应中的某个位置包含该字符串。这称为通用查询，是探测 URL 是否对应于 GraphQL 服务的有用工具

- 通用端点名称：GraphQL 服务通常使用类似的端点后缀。测试 GraphQL 端点时，您应该尝试将通用查询发送到这些路径（参见常见API路径）（如果这些常见端点没有返回 GraphQL 响应，可尝试附加`/v1`到路径

  

   

###### 信息泄露/越权

不当的配置可能导致网站的graphql架构以及存储的敏感数据暴露

有的API使用内省查询可以列出列出GraphQL中所有Query、Mutation、ObjectType、Field、Arguments



返回包返回的就是该API端点的所有信息



放到[GraphQL Voyager](https://graphql-kit.com/graphql-voyager/)中可以生成可视化的文档



在列出的信息中，寻找敏感信息(比如email、password、secretKey、token、licenseKey、session；还可以多关注废弃字段deprecated fields；还有remove等增删改查的功能)



使用找到的敏感信息等构造请求查询，如果权限校验不当，就会造成越权或者未授权查询





###### GraphQL注入

所谓注入，无非就是构造闭合并插入其他语句执行，这个漏洞P神也在先知白帽大会分享过，一看就懂（红色部分为用户可控的注入数据）

![image-20250311205909641](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311205909641.png)



漏洞的出现场景大致为graphql语句不可控，但语句中的部分参数用户可控，逻辑大致为：

用户访问URL -> 前端获取参数 -> 拼接成GraphQL语句 -> 发送 -> 后端执行

在这种情况下，就可以尝试GraphQL注入改变原本的GraphQL语义进行漏洞利用



和SQL类似，防御这类漏洞使用参数化查询即可：通过独立的参数（如JSON）传入变量值，而非直接拼接字符串。







###### SQL注入等传统漏洞





###### 相关工具的前端漏洞

GraphiQL（一个浏览器GraphQL客户端，纯前端应用）及使用其的Graphene-Django框架可能会产生一些前端漏洞比如CSRF、click hijacking等









### 1.访问私有帖子

[Accessing private GraphQL posts](https://portswigger.net/web-security/graphql/lab-graphql-reading-private-posts)

本实验的博客页面包含一个隐藏的博客帖子，该帖子具有秘密密码。要解决实验室，找到隐藏的博客文章，并输入密码。



从数据包中可以看到使用了GraphQL

返回包中返回了每个帖子的信息，包括id等，但是发现四篇博客id分别为1，2，4，5，少了3：

![image-20250311220043229](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311220043229.png)



把这个查询的数据包发送到repeater中



然后会出现Graphql，点击这个tab(不点也行)，可以直接在最开始的数据包中修改：

修改id为3，3为隐藏文章

![image-20250311220803844](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311220803844.png)











然后点击下面这个：**用`BP`自带的内省查询进行测试**，发现响应中有`postPassword`字段

![image-20250311220315781](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311220315781.png)

![image-20250311221504072](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311221504072.png)





回到原先的查询字段，将响应中的`postPassword`字段放到下面请求包中：

![image-20250311223220415](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311223220415.png)





复制密码，提交：

![image-20250311223300312](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250311223300312.png)







### 2.私有GraphQL字段的意外暴露

[Accidental exposure of private GraphQL fields](https://portswigger.net/web-security/graphql/lab-graphql-accidental-field-exposure)

本实验的用户管理功能由GraphQL端点提供支持。本实验包含一个访问控制漏洞，您可以通过该漏洞诱导API显示用户凭据字段。
要解决实验，请以管理员身份登录并删除用户名carlos。



点击My account，登录抓包，发现Graphql的api接口，发送进行自省：

![image-20250312124026604](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312124026604.png)





右键发送到`site map`，查看：



![image-20250312124130159](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312124130159.png)



发现一个api的查询`getUser`，通过传递id返回用户名密码：

![image-20250312124326865](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312124326865.png)



修改为1，应该是管理员，然后登录，手动删除指定账户

![image-20250312124455053](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312124455053.png)

![image-20250312124539923](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312124539923.png)





### 3.寻找隐藏的GraphQL端点

[Finding a hidden GraphQL endpoint](https://portswigger.net/web-security/graphql/lab-graphql-find-the-endpoint)



本实验的用户管理功能由一个隐藏的GraphQL端点提供支持。您将无法通过简单地单击站点中的页面来找到此端点。端点也有一些针对内省的防御措施。
要解决实验室，找到隐藏的端点，并删除卡洛斯。







在Repeater中，向一些常见的GraphQL端点后缀发送请求并检查结果。

![image-20250312125325522](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312125325522.png)



当向`/api`发送GET请求时，响应包含“Query not present”错误。这暗示在这个位置可能有一个GraphQL端点响应GET请求。



修改请求参数：

![image-20250312125443388](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312125443388.png)





继续修改get的参数为内省：

```
/api?query=query+IntrospectionQuery+%7B%0A++__schema+%7B%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A

```

发现是不允许内省的，被拦截

![image-20250312125621689](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312125621689.png)





修改查询在`__schema`后包含一个换行符(`%0a`)绕过检测，并重新发送：

```
/api?query=query+IntrospectionQuery+%7B%0D%0A++__schema%0a+%7B%0D%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A

```

响应现在包含了完整的内省细节。

这是因为服务器被配置为排除与正则表达式`__schema{`匹配的查询，即使查询仍然是有效的内省查询，查询也不再匹配。



然后右键请求包， 选择GraphQL > Save GraphQL queries to site map：

![image-20250312130012836](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312130012836.png)

回到Target看一下，找到一个`getUser`的查询

![image-20250312130553400](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312130553400.png)

查出对应用户的id：

![image-20250312130729736](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312130729736.png)



还有一个`deleteOrganizationUser`的mutation（变更），需要传递id



修改id为3，删除用户

```
    /api?query=mutation+%7B%0A%09deleteOrganizationUser%28input%3A%7Bid%3A+3%7D%29+%7B%0A%09%09user+%7B%0A%09%09%09id%0A%09%09%7D%0A%09%7D%0A%7D

```

![image-20250312130830594](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312130830594.png)









### 4.使用别名绕过GraphQL暴力破解保护

[Bypassing GraphQL brute force protections](https://portswigger.net/web-security/graphql/lab-graphql-brute-force-protection-bypass)

本实验的用户登录机制由GraphQL API提供支持。API端点有一个速率限制器，如果它在短时间内从同一来源接收到太多请求，则会返回错误。
要解决这个实验，请强制登录机制以卡洛斯身份登录。使用身份验证实验室密码列表作为密码源。





>  许多端点都会设置某种速率限制器来防止暴力攻击。一些速率限制器基于收到的 `HTTP` 请求数而不是在端点上执行的操作数来工作。由于别名实际上允许您在单个 `HTTP` 消息中发送多个查询，因此它们可以绕过此限制。



抓登录的数据包，可以看到graphql接口

可以看到是通过“变更”实现的登录：

![image-20250312131653619](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312131653619.png)



在GraphQL选项卡中，创建一个使用别名在一条消息中发送多个登录变化的请求。

可以使用靶场中的提示在浏览器中快速生成

![image-20250312132326423](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312132326423.png)



响应中返回了每次登录结果，找出为`true`的，使用对应的账户密码登录`carlos`的账户



![image-20250312132521022](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312132521022.png)







### 5.通过GraphQL执行CSRF漏洞利用

[Performing CSRF exploits over GraphQL](https://portswigger.net/web-security/graphql/lab-graphql-csrf-via-graphql-api)

本实验的用户管理功能由GraphQL端点提供支持。端点接受内容类型为x-www-form-urlencoded的请求，因此容易受到跨站点请求伪造（CSRF）攻击。
为了解决这个实验，制作一些HTML，使用CSRF攻击来更改查看者的电子邮件地址，然后将其上传到您的漏洞利用服务器。
您可以使用以下凭据登录到自己的帐户：wiener：peter。



当 `GraphQL` 端点未验证发送给它的请求的内容类型且未实现 `CSRF` 令牌时，可能会出现 `CSRF` 漏洞。

只要内容类型经过验证，使用内容类型的 `POST` 请求`application/json`就不会被伪造。在这种情况下，即使受害者访问了恶意网站，攻击者也无法让受害者的浏览器发送此请求。

但是，浏览器可以发送诸如 `GET` 之类的替代方法或任何内容类型为`x-www-form-urlencoded`的请求，因此如果端点接受这些请求，用户可能会容易受到攻击。在这种情况下，攻击者可能能够利用漏洞向 `API` 发送恶意请求





登录账户，点击修改电子邮件，抓包，可以看到是Graphql的“变更”实现的，发送到repeater：

![image-20250312133003819](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312133003819.png)





修改POST请求为`Content-Type` of `x-www-form-urlencoded`

点击两次修改：

![image-20250312133227774](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312133227774.png)

此时POST请求体已经被删除了，我们使用url编码重新添加

```
    query=%0A++++mutation+changeEmail%28%24input%3A+ChangeEmailInput%21%29+%7B%0A++++++++changeEmail%28input%3A+%24input%29+%7B%0A++++++++++++email%0A++++++++%7D%0A++++%7D%0A&operationName=changeEmail&variables=%7B%22input%22%3A%7B%22email%22%3A%22hacker%40hacker.com%22%7D%7D

```



然后生成csrf poc并保存为html



网页上点击Go to exploit server，把生成的html粘贴过去，然后发送exp到受害者

![image-20250312133557721](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250312133557721.png)



### 参考文章

都是通过看各位师傅的文章学习的，感谢各位师傅的文章：

https://mdnice.com/writing/45b3110a67214cb589374de8c854c846

https://blog.csdn.net/weixin_58370748/article/details/139563980

https://www.cnblogs.com/yuy0ung/articles/18706771

https://mp.weixin.qq.com/s?__biz=Mzg3OTEwMzIzNA==&mid=2247483810&idx=1&sn=725fd1283f3d411a8925b4ffcc076b45&scene=21#wechat_redirect

burp靶场：https://portswigger.net/web-security/all-labs#graphql-api-vulnerabilities

DVGA靶场：https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/burp%E5%AE%98%E6%96%B9graphql-api%E9%9D%B6%E5%9C%BA%E8%AE%B0%E5%BD%95/  

