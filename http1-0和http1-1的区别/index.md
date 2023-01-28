# http1.0和http1.1和http2的区别






## HTTP简介

HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。HTTP是一个基于TCP/IP通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）





## HTTP1.0和1.1的差别

### 1.长连接
长连接（HTTP persistent connection ，也有翻译为持久连接），指数据传输完成了保持TCP连接不断开（不发RST包、不四次握手），等待在同域名下继续用这个通道传输数据；相反的就是短连接。
**HTTP1.1支持长连接和请求的流水线**（Pipelining）处理，并且默认使用长连接，如果加入"Connection: close "，才关闭。
**HTTP 1.0默认使用短连接**，规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个TCP连接，服务器完成请求处理后立即断开TCP连接，服务器不跟踪，每个客户也不记录过去的请求。要建立长连接，可以在请求消息中包含Connection: Keep-Alive头域，如果服务器愿意维持这条连接，在响应消息中也会包含一个Connection: Keep-Alive的头域。



### 2.HOST域
**HTTP1.1在Request消息头里头多了一个Host域，而且是必传的，HTTP1.0则没有这个域。**
在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发 展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。

HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。此外，服务器应该接受以绝对路径标记的资源请求。



### 3.带宽优化
HTTP/1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了。又比如下载大文件时不支持断点续传功能，在发生断连后不得不重新下载完整的包。

**HTTP/1.1中在请求消息中引入了range头域，它支持只请求资源的某个部分**。在响应消息中Content-Range头域声明了返回的这部分对象的偏移值和长度。如果服务器相应地返回了对象所请求范围的内容，则响应码为206（Partial Content），它可以防止Cache将响应误以为是完整的一个对象。

Range头域可以请求实体的一个或者多个子范围。例如

```
表示头500个字节：bytes=0-499
表示第二个500字节：bytes=500-999
表示最后500个字节：bytes=-500
表示500字节以后的范围：bytes=500-
第一个和最后一个字节：bytes=0-0,-1
同时指定几个范围：bytes=500-600,601-999
```

Content-Range： 表示WEB服务器传送的范围，描述响应覆盖的范围和整个实体长度。一般格式为：bytes-unitSPfirst-byte-pos-last-byte-pos/entity-legth

例 如，传送头500个字节次字段的形式：

```
Content-Range:bytes0- 499/1234 
```

上述的1234为整个实体的长度。



另外一种浪费带宽的情况是请求消息中如果包含比较大的实体内容，但不确定服务器是否能够接收该请求（如是否有权限），此时若贸然发出带实体的请求，如果被拒绝也会浪费带宽。

HTTP/1.1加入了一个新的状态码100（Continue）。客户端事先发送一个只带头域的请求，如果服务器因为权限拒绝了请求，就回送响应码401（Unauthorized）；如果服务器接收此请求就回送响应码100，客户端就可以继续发送带实体的完整请求了。具体用法为：客户端在Request头部中包含Expect: 100-continue
Server看到之后呢如果回100 (Continue) 这个状态代码，客户端就继续发requestbody。(注意，HTTP/1.0的客户端不支持100响应码，这个是HTTP1.1才有的。）如果回401，客户端就知道是什么意思了。



### 4.请求方法和响应状态码
HTTP1.1增加了`OPTIONS,PUT, DELETE, TRACE, CONNECT`这些Request方法
HTTP1.1 增加的新的status code有:
(HTTP1.0没有定义任何具体的1xx status code, HTTP1.1有2个)

```
100 Continue
101 Switching Protocols

203 Non-Authoritative Information
205 Reset Content
206 Partial Content

302 Found (在HTTP1.0中有个 302 Moved Temporarily)
303 See Other
305 Use Proxy
307 Temporary Redirect

405 Method Not Allowed
406 Not Acceptable
407 Proxy Authentication Required
408 Request Timeout
409 Conflict
410 Gone
411 Length Required
412 Precondition Failed
413 Request Entity Too Large
414 Request-URI Too Long
415 Unsupported Media Type
416 Requested Range Not Satisfiable
417 Expectation Failed

504 Gateway Timeout
505 HTTP Version Not Supported
```



### 5.Cache (缓存)
**在HTTP/1.0中，已经定义不少有关缓存的头域：**

- **Expires：**浏览器会在指定过期时间内使用本地缓存，指明应该在什么时候认为文档已经过期，从而不再缓存它，时间为格林威治时间GMT。例如: Expires: Thu, 19 Nov 1981 08:52:00 GMT

- **Last-Modified**：请求对象最后一次的修改时间，用来判断缓存是否过期 通常由文件的时间信息产生

- **Date**：生成消息的具体时间和日期，即当前的GMT时间。例如：　Date: Sun, 17 Mar 2013 08:12:54 GMT

- **If-Modified-Since**：客户端存取的该资源最后一次修改的时间，用来和服务器端的Last-Modified做比较

- **Set-Cookie**: 用于把cookie 发送到客户端。例如: `Set-Cookie:PHPSESSID=c0huq7pdkmm5gg6osoe3mgjmm3; path=/`

- **Pragma:no-cache**：客户端使用该头域说明请求资源不能从cache中获取，而必须回源获取。

  

**HTTP/1.1在1.0的基础上加入了一些cache的新特性：**

1.当缓存对象的Age超过Expire时变为stale对象，cache不需要直接抛弃stale对象，而是与源服务器进行重新激活（revalidation）。

2.为了使caching机制更加灵活，HTTP/1.1增加了**Cache-Control**头域（请求消息和响应消息都可使用），它支持一个可扩展的指 令子集。请求时的缓存指令包括no-cache、no-store、max-age、max-stale、min-fresh、only-if- cached，
响应消息中的指令包括public、private、no-cache、no-store、no-transform、must- revalidate、proxy-revalidate、max-age。各个消息中的指令含义如下：
**Public**指示响应可被任何缓存区缓存,并且在多用户间共享。
Private指示对于单个用户的整个或部分响应消息，不能被共享缓存处理，此响应消息对于其他用户的请求无效。
**no-cache**指示请求或响应消息不能缓存
**no-store**用于防止重要的信息被无意的发布，在请求消息中发送将使得请求和响应消息都不使用缓存。
**max-age**指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。
**min-fresh**指示客户机可以接收响应时间小于当前时间加上指定时间的响应。
**max-stale**指示客户机可以接收超出超时期间的响应消息。
**must-revalidate**：如果数据是过期的则去服务器进行获取

而且在请求消息或响应消息中设置Cache-Control并不会修改另一个消息处理过程中的缓存 处理过程。

3.Cache使用关键字索引在磁盘中缓存的对象：在HTTP/1.0中使用资源的URL作为关键字，但可能存在不同的资源基于同一个URL的情况，要区别它们还需要客户端提供更多的信息，如Accept-Language和Accept-Charset头域。为了更好的支持这种内容协商机制(content negotiation mechanism)，**HTTP/1.1在响应消息中引入了Vary头域，该头域列出了请求消息中需要包含哪些头域用于内容协商。**例如: Vary: Accept-Encoding

```
请求头字段 说明 响应头字段
Accept 告知服务器发送何种媒体类型 Content-Type
Accept-Language 告知服务器发送何种语言 Content-Language
Accept-Charset 告知服务器发送何种字符集 Content-Type
Accept-Encoding 告知服务器采用何种压缩方式 Content-Encoding
```

有时候，上面四个 Accept 字段并不够用，例如要针对特定浏览器如 IE6 输出不一样的内容，就需要用到请求头中的 User-Agent 字段。类似的，请求头中的 Cookie 也可能被服务端用做输出差异化内容的依据。由于客户端和服务端之间可能存在一个或多个中间实体（如缓存服务器），而缓存服务最基本的要求是给用户返回正确的文档。如果服务端根据不同 User-Agent 返回不同内容，而缓存服务器把 IE6 用户的响应缓存下来，并返回给使用其他浏览器的用户，肯定会出问题 。所以 HTTP 协议规定，如果服务端提供的内容取决于 User-Agent 这样常规 Accept 协商字段之外的请求头字段，那么响应头中必须包含 Vary 字段，且 Vary 的内容必须包含 User-Agent。同理，如果服务端同时使用请求头中 User-Agent 和 Cookie 这两个字段来生成内容，那么响应中的 Vary 字段看上去应该是这样的：`Vary: User-Agent, Cookie`
也就是说 Vary 字段用于列出一个响应字段列表，告诉缓存服务器遇到同一个 URL 对应着不同版本文档的情况时，如何缓存和筛选合适的版本。





## http1和http2区别

HTTP/2（超文本传输协议第2版，最初命名为HTTP 2.0），是HTTP协议的的第二个主要版本，使用于万维网。HTTP/2是HTTP协议自1999年HTTP 1.1发布后的首个更新，主要基于SPDY协议（是Google开发的基于TCP的应用层协议，用以最小化网络延迟，提升网络速度，优化用户的网络使用体验。



### HTTP/1.1的缺陷
#### 1.高延迟

> 带来页面加载速度的降低网络延迟问题主要由于队头阻塞(Head-Of-Line Blocking),导致带宽无法被充分利用。

    队头阻塞是指当顺序发送的请求序列中的一个请求因为某种原因被阻塞时，在后面排队的所有请求也一并被阻塞，会导致客户端迟迟收不到数据。

#### 2.无状态特性

> 带来的巨大HTTP头部

由于报文Header一般会携带"User Agent""Cookie""Accept""Server"等许多固定的头字段，多达几百字节甚至上千字节，但Body却经常只有几十字节（比如GET请求、
204/301/304响应），成了不折不扣的“大头儿子”。Header里携带的内容过大，在一定程度上增加了传输的成本。更要命的是，成千上万的请求响应报文里有很多字段值都是重复的，非常浪费。



#### 3.明文传输--带来的不安全性
HTTP/1.1在传输数据时，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份，这在一定程度上无法保证数据的安全性。

#### 4.不支持服务器推送消息



### HTTP/2 新特性
#### 1.二进制传输
HTTP/2传输数据量的大幅减少,主要有两个原因:以二进制方式传输和Header 压缩。我们先来介绍二进制传输,HTTP/2 采用二进制格式传输数据，而非HTTP/1.x 里纯文本形式的报文 ，二进制协议解析起来更**高效**,**错误更少**。 HTTP/2 将请求和响应数据分割为更小的帧，并且它们采用二进制编码。

#### 2.Header 压缩
HTTP/2并没有使用传统的压缩算法，而是开发了专门的"HPACK”算法，在客户端和服务器两端建立“字典”，用索引号表示重复的字符串，还采用哈夫曼编码来压缩整数和字符串，可以达到50%~90%的高压缩率。

```
假定一个页面有80个资源需要加载（这个数量对于今天的Web而言还是挺保守的）, 而每一次请求都有1400字节的消息头（着同样也并不少见，因为Cookie和引用等东西的存在）, 至少要7到8个来回去“在线”获得这些消息头。这还不包括响应时间——那只是从客户端那里获取到它们所花的时间而已。这全都由于TCP的慢启动机制，它会基于对已知有多少个包，来确定还要来回去获取哪些包 – 这很明显的限制了最初的几个来回可以发送的数据包的数量。相比之下，即使是头部轻微的压缩也可以是让那些请求只需一个来回就能搞定——有时候甚至一个包就可以了。这种开销是可以被节省下来的，特别是当你考虑移动客户端应用的时候，即使是良好条件下，一般也会看到几百毫秒的来回延迟。
```



#### 3.多路复用
在 HTTP/2 中引入了多路复用的技术。多路复用很好的解决了浏览器限制同一个域名下的请求数量的问题，同时也接更容易实现全速传输，毕竟新开一个 TCP 连接都需要慢慢提升传输速度。

```
HTTP/1.x 有个问题叫线端阻塞(head-of-line blocking), 它是指一个连接(connection)一次只提交一个请求的效率比较高, 多了就会变慢。 HTTP/1.1 试过用流水线(pipelining)来解决这个问题, 但是效果并不理想(数据量较大或者速度较慢的响应, 会阻碍排在他后面的请求). 此外, 由于网络媒介(intermediary )和服务器不能很好的支持流水线, 导致部署起来困难重重。而多路传输(Multiplexing)能很好的解决这些问题, 因为它能同时处理多个消息的请求和响应; 甚至可以在传输过程中将一个消息跟另外一个掺杂在一起。所以客户端只需要一个连接就能加载一个页面。
```



#### 4.Server Push
HTTP2还在一定程度上改变了传统的“请求-应答”工作模式，服务器不再是完全被动地响应请求，也可以新建“流”主动向客户端发送消息。比如，在浏览器刚请求HTML的时候就提前把可能会用到的JS、CSS文件发给客户端，减少等待的延迟，这被称为"服务器推送"（ Server Push，也叫 Cache push）

```
当浏览器请求一个网页时，服务器将会发回HTML，在服务器开始发送JavaScript、图片和CSS前，服务器需要等待浏览器解析HTML和发送所有内嵌资源的请求。服务器推送服务通过“推送”那些它认为客户端将会需要的内容到客户端的缓存中，以此来避免往返的延迟。
```



#### 5.提高安全性
出于兼容的考虑，HTTP/2延续了HTTP/1的“明文”特点，可以像以前一样使用明文传输数据，不强制使用加密通信，不过格式还是二进制，只是不需要解密。















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/http1-0%E5%92%8Chttp1-1%E7%9A%84%E5%8C%BA%E5%88%AB/  

