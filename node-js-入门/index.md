# Node.js 入门




Node.js是一个机遇Chrome V8引擎的JavaScript运行环境

Node.js的包管理器npm，是全球最大的开源生态库系统

Node.js可解析js代码，没有浏览器安全级别的限制



## 一.开发环境配置

官网： https://nodejs.org/en/

API文档：http://nodejs.cn/api/events.html

#### 如果需要安装多个版本

**1).docker**

**2).使用nvm来安装并维护（`推荐`）**

项目地址：https://github.com/nvm-sh/nvm

```bash
# 1.安装nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

# 2.配置环境变量
#linux下  .bash_profile或.bash_rc
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm

# windows系统。。。
```

3.配置加速镜像：

```bash
# 同样在上面的配置文件
export NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node
```



#### **安装node**

```bash
# 安装指定版本
nvm install 8.0.0

# 或安装最新的稳定版本
nvm install --lts

# 或安装最新的版本
nvm install node

# 查看安装的版本
nvm ls

# 查看当前版本
node -v

# 切换版本
nvm use 6.11.2
```



#### 体验一下

在终端进入node命令行：`node`, 之后可以输入js代码了

在终端里面，不用受到浏览器里面的一些限制(但是本地环境没有`dom`和`bom`的操作)

比如，查看本地进程`process`

执行文件：`node index.js`



###### 不用每次写完都运行命令来执行

安装`nodemon`, 实时侦测文件的变化

```bash
npm install nodemon -g  # 全局安装
```

npm使用国内源：

```bash
# 永久
npm config set registry https://registry.npm.taobao.org
```

或安装使用`cnpm`命令

使用`nodemon`：

```bash
nodemon index.js
# 这样就可以实时侦测了
```



使用其他版本node来运行脚本

```bash
# 1.指定版本来运行
nvm run 6.1.2 index.js


#2.使用配置文件
.nvmrc 
#内容直接写版本号，
6.1.2
#
nmp use
#接下来运行就可以了
node index.js
```



## 二. 模块(包)与CommonJS

![](http://image.xpshuai.cn/commonjs.png)

#### Node.js三种模块

1. **内置的Node.js模块**

   ```javascript
   const os = require('os')  // require 载入模块
   console.log(os.hostname()); // 显示主机名
   ```

   

2. **第三方的Node.js模块**(https://www.npmjs.com)

   ​	把包用npm引进来（可以创建配置文件来管理`npm init `,根据情况填写回撤，会自动创建`package.json`）

   ​    在命令行输入`npm install request  --save` 安装在配置信息的依赖`dependencies`里

   ​    会在我们的包里面自动创建一个文件夹`node_modules`, 我们安装的包就会在里面了

   ​	载入模块同上

   ​	然后开始使用第三方模块

   ```javascript
   const request = require('request')  //自动先从自己的模块找，找不到就会到我们目录下配置文件夹下找
   
   request({
     url: 'https://api.douban.com/v2/movies/top250',
     json:true,
   }, (error, response, body) => {
     console.log(JSON.stringfy(body, null, 2));
   
   });
   
   ```

   

3. **自定义的Node.js模块（要按照CommonJS规范）**

   新建一个目录`src`,里面新建一个js文件(模块)

   ```javascript
   //自定义模块
   const hello = () =>{
     console.log("hello~");
   }
   
   //暴露接口供外部使用(第一个hello是模块的名字，第二个hello是要用的方法)
   module.exports.hello = hello
   ```

   调用

   ```javascript
   const greet = require("./src/greeting.js")
   greet.hello()
   ```

   



## 三. NPM-包管理工具

#### npm使用

```bash
npm -v
npm info
```



**镜像源**

```bash
# 查看
npm config get registry

# 设置
npm config set registry http://registry.npmjs.org 
```



**升级node**

```bash
#
npm install n -g   
```



**查看当前目录下安装了哪些node包**

```bash
npm ls
```

**查看当前npm用户**

```bash
npm whoami
```

**npm清理缓存**

```bash
npm cache clean -f

```



**npm安装模块**

```bash
# i = install
npm i forever -g  # 全局

npm install xxx   # 利用 npm 安装xxx模块到当前命令行所在目录；
npm install -g xxx  # 利用npm安装全局模块xxx；
npm install xxx  # 安装但不写入package.json；
npm install xxx –save  # 安装并写入package.json的”dependencies”中；
npm install xxx –save-dev # 安装并写入package.json的”devDependencies”中。
```

**npm 删除模块**

```bash
npm uninstall xxx     # 删除xxx模块；
npm uninstall -g xxx  # 删除全局模块xxx；
```



**更新模块**

```
npm update 
```

**检查模块是否已经过时**

```bash
npm outdated
```

在项目中引导创建一个package.json文件

```bash
npm init
```

 发布模块

```bash
npm publish
```

查看node安装路径

```bash
查看node安装路径
```



**npm scripts**

配置文件中

```json
  "scripts": {
    "test": "echo \"Error: no test sspecified\" && exit 1"
    "build": "node async.js"
  },
```

按照上面配置完之后，运行只需：`npm run build`

优点：

- 功能相同，可以使用同一个脚本

详细的下来查文档



## 四.深入浅出Node核心模块API

#### 1). url

> 记得要导入模块

###### 1.parse

`url.parse(urlStr, [parseQueryString], [slashesDenoteHost])`

```javascript
url.parse("https://www.baidu.com")
# 井号后面是hash
url.parse("https://www.baidu.com:8080/api.php?from=xp&course=node#level")

#第二个参数 parseQueryString 为true时将使用查询模块分析查询字符串，默认为false
url.parse("https://www.baidu.com"， true)
#第三个参数 如果设置成true，//foo/bar 形式的字符串将被解释成 { host: ‘foo', pathname: ‘/bar' }
url.parse("https://www.baidu.com"， true,true)
```

###### 2.format

将一个解析后的URL对象、转成、一个格式化的URL字符串。

`url.format(urlObj)`

```javascript
var url = require('url');
 
var a = url.format({
protocol : 'http' ,
auth : null ,
host : 'example.com:8080' ,
port : '8080' ,
hostname : 'example.com' ,
hash : null ,
search : '?a=index&t=article&m=default',
query : 'a=index&t=article&m=default',
pathname : '/one',
path : '/one?a=index&t=article&m=default',
href : 'http://example.com:8080/one?a=index&t=article&m=default'
});
console.log(a);
 
//输出结果：http://example.com:8080/one?a=index&t=article&m=default
```



###### 3.resolve

为URL或 href 插入 或 替换原有的标签，两段解析成一个完整的url

`url.resolve(from, to)`

```javascript
var url = require('url');
var a = url.resolve('/one/two/three', 'four') ,
b = url.resolve('http://example.com/', '/one'),
c = url.resolve('http://example.com/one', '/two');
console.log(a +","+ b +","+ c);
//输出结果：
///one/two/four
//http://example.com/one
//http://example.com/two
```



#### 2). QueryString

###### 1.querystring.escape(str)

转义为url编码

```javascript
querystring.escape('<大家好>')
```

###### 2.querystring.unescape(str)

反转义

```javascript
querystring.unescape('%3C%E5%A4%A7%E5%AE%B6%E5%A5%BD%3E')
```



###### 3.querystring.stringify(obj,[, seq[, eq[, options]]])

序列化

```javascript
1.  querystring.stringify({name:'chenshuai',ago:21,job:"web"})
2.   querystring.stringify({name:'chenshuai',ago:21,job:"web"},".") //第二个参数可以修改连接的&

3. querystring.stringify({name:'chenshuai',ago:21,job:"web"},".",":")//第三个参数可以修改key与value之间的字符串

```



###### 4.querystring.parse(str[, sep[, eq[, options]]])

反序列化

```javascript
// querystring.parse('字符串','&','=') //默认状态(字符串,分隔符,键与值)
1. querystring.parse('name=chenshuai&ago=21&job=web')

2.querystring.parse('name=chenshuai.ago=21.job=web','.')//返序列化时要注意格式

3.querystring.parse('name:chenshuai.ago:21.job:web','.',':')
```



#### 3).http(s) 模块

###### 1.get方法

案例，http小爬虫

```javascript
var http = require('http')
var https = require('https') //https网站使用(和http的方法是一样的)
var cherio = require('cherio') // 类似于jQuery
var url = "https://www.lagou.com"

// 解析源代码
function parseMenu(html){
  //先导入html，然后再进行处理
  var $ = cherio.load(html)
  var menu = $('.category-list')
  var menuData = []
  menu.each(function(index,value){ //遍历
    var menuTitle = $(value).find('h2').text() //一级标题
    var menuLists = $(value).find('a')
    var menuList = []
    menuLists.each(function(inde, val){ //遍历
      menuList.push($(val).text())  //a元素的text
    })
    menuData.push({
      menuTitle:menuTitle,
      menuList:menuList
    })
  })
  return menuData
}

//打印
function printMenu(menu)
{
  menu.forEach(function(val){
    console.log(val.menuTitle + '\n');
    val.menuList.forEach(function(value){
      console.log(value);
    })
  })

}

// get请求
https.get(url, function(res){
  var html = '';
  res.on('data', function(data){
    html += data;
  })

  res.on('end', function(){
    // 打印的是网页源代码
    //console.log(html);
    var result = parseMenu(html)
    printMenu(result)

  })

  res.on('error', function(err){
    console.log(err);
  })
})

```



###### 2.Request方法

GET获取异步数据

```javascript
//Http模块的request方法的get来获取异步数据 豆瓣电影Top250
const https = require('https')

var options = {
  hostname: 'douban.uieee.com',
  port: 443,
  method: 'GET',
  path: '/v2/movie/top250'
}

var responseData = ''
var request = https.request(options, (response) =>{
  response.setEncoding('utf8')
  response.on('data', (chunk)=>{
    responseData += chunk
  })
  response.on('end',()=>{
    JSON.parse(responseData).subjects.map((item) =>{
      console.log(item.title); // 第一页的电影标题
    })
  })
})

request.on('error', (error)=>{
  console.log(error);
})

request.end()

```

POST方法提交表单

```javascript
//Http模块的request方法的post方法提交表单
const http = require('http')
const qs = require('querystring')

//序列化
var postData = querystring.stringfy({
  'question[title]':'哈哈哈',
  'question[content]':'我的内容'
})

var options = {
  hostname: 'www.codingke.com',
  port: 80,
  method: 'POST',
  path: '/ajax/course/question'
  headers: {  // 因为登录之后才能提交，所有这里...
    'Cookie':'xxxx',
    'User-Agent':'xxxx',
    'xxx':'xxx',
    'Content-Length':postData.length
  }
}


var request = http.request(options, (res) =>{
  console.log('Status:' + res.statusCode);
  res.setEncoding('utf8')
  res.on('data', (chunk)=>{
      console.log(chunk);
  })

  res.on('end',()=>{
    console.log("表单提交完毕");
  })
})

//提交表单
request.write(postData)


request.on('error', (error)=>{
  console.log(error);
})

request.end()

```



#### 4).  Events事件模块

使用事件: `EventEmitter`类

事件的参数

只执行一个的事件监听器

比如：更新、发布 都是一个事件



小例子:

```javascript
const EventEmitter = require('events')

class Player extends EventEmitter{}  //继承类
//实例化对象
var player = new Player()

//自定义一个函数(不过我这里是用的匿名函数)
function ev1(){
    console.log("嘻嘻");
}

//监听事件(这里用的匿名函数)
player.on('play', (track)=>{
  console.log(`正在播放: 《${track}》`);
})

// emit订阅事件，传递参数
player.emit('play',"被风吹过的夏天")
//事件定义之后，可以多次触发
player.emit('play',"朋友请听好")

// once 不论你定义几次，只会触发一次
player.once('play',"陈情令")

// removeListener 移除指定的监听事件
//player.removeListener('')


// removeAllListeners    移除所有的监听事件

// eventNames方法可以罗列出已注册的所有监听器的事件，类型为数组

```



#### 5). fs 文件系统

得到文件与目录信息：`stat`

创建一个目录：`mkdir`

创建文件并写入内容：`writeFile`,`appendFile`

读取文件的内容：`readFile`

列出目录的东西：`readdir`

重命名目录或文件：`rename`

删除目录与文件：`rndir`,`unlink`

```javascript
const fs = require('fs')

fs.stat('index.js', (error,stats) => {
  if(error){
    console.log(error);
  }else{
    console.log(stats);  //打印文件的一些信息
    console.log(`文件：${stats.isFile()}`);
    console.log(`目录：${stats.isDirectory()}`);
  }
})


//新建目录
fs.mkdir('logs' (error) =>{
  if(error){
    console.log(error);
  }else{
    console.log("成功创建目录: logs");
  }
})

// 往文件写内容(如果文件不存在，就先创建)
fs.writeFile('logs/hello.log', "hello ~\n" , (error) =>{
  if(error){
    console.log(error);
  }else{
    console.log("成功向文件logs/hello.log写入内容 ");
  }
})

// 修改文件内容
fs.writeFile('logs/hello.log', "你好 ~\n" , (error) =>{
  if(error){
    console.log(error);
  }else{
    console.log("成功修改文件logs/hello.log内容 ");
  }
})

// 追加文件内容
fs.appendFile('logs/hello.log', "新添加的一行 ~\n" , (error) =>{
  if(error){
    console.log(error);
  }else{
    console.log("成功添加内容文件logs/hello.log ");
  }
})

// 读取文件内容
fs.readdFile('logs/hello.log','utf8', (error,data) =>{
  if(error){
    console.log(error);
  }else{
    console.log("成功读取文件logs/hello.log内容");
    console.log(data);
    //console.log(data.toString());
  }
})


// 读取文件夹列表
fs.readdir('logs/', (error,data) =>{
  if(error){
    console.log(error);
  }else{
    console.log("成功读取文件夹logs/列表 ");
    console.log(data);
    //console.log(data.toString());
  }
})

//对文件重命名  、修改文件夹名字一样
fs.rename('logs/hello.log', "logs/hello_new.log" , (error) =>{
  if(error){
    console.log(error);
  }else{
    console.log('重命名文件名logs/hello.log ");
  }
})

//循环删除文件
fs.readdirSync('logs').map((file) =>{
  fs.unlink(`logs/${file}`, (error) =>{
    if(error){
      console.log(error);
    }else{
      console.log(`成功删除文件logs/${file}`);
    }
  })
})



//删除文件夹
fs.rmdir('logs',  (error) =>{
  if(error){
    console.log(error);
  }else{
    console.log('成功删除目录:logs/ ");
  }
})

```



#### 6). Stream

文件流

```javascript
const fs = require('fs')

var fileReadStrem = fs.createReadStream('data.json')
var fileWriteStrem = fs.createWriteStream('data1.json')

var count = 0;

//读取文件流
fileReadStrem.once('data', (chunk) => {
  console.log(chunk.toString());
})

fileReadStrem.on('data', (chunk) => {
  console.log(`${ ++count } 接收到：${chunk.length}`);
  //写入文件流
  fileWriteStrem.write(chunk)
})

fileReadStrem.on('end', () => {
  console.log("-- 结束  --");
})

fileReadStrem.on('error', (error) => {
  console.log(error);
})

```

pipe 

```javascript
const fs = require('fs')
const zlib = require('zlib')

var fileReadStrem = fs.createReadStream('data.json')
var fileWriteStrem = fs.createWriteStream('data1.json')

fileWriteStrem.on('pipe', (source) => {
  console.log(source); //将写入的内容通过管道打印一下
})

//读取文件流的内容放到文件写入流   管道
fileReadStrem
  .pipe(zlib.createGzib())
  .pipe(fileWriteStrem)

```





## 五.使用Node创建后端路由

**使用supervisor运行**

```bash
# 安装：
npm install supervisor -g

# 运行
supervisor router_1.js
```



**最简单路由**

router_1.js

```javascript
var http = require('http')
var url = require('url')

var route = require('./modules/route.js')

http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type':'text/html;charset=utf-8'})
  if(req.url !== '/favicon.ico'){
    var pathName = url.parse(req.url).pathname.replace(/\//,'') //http://localhost:8000/login就是login
    console.log(pathName);
    try{  //最简单的异常处理
      route[pathName](req, res)
    } catch(err){
      route['home'](req, res)  //默认路由，如果不存在哪个路由，就返回这个路由
    }

  }
  res.end()
}).listen(8000)

console.log('Server running at http://localhost:8000');

```

route.js

```javascript
module.exports = {
  home: (req, res) => {
    res.write("欢迎来到首页")  // 默认路由
  },
  login: (req, res) => {
    res.write("登录界面")  // 处理路由
  },
  registor: (req, res) => {
    res.write("注册界面")  // 处理路由
  }
}

```



**读取图片**

router_02.js

```javascript
var http = require('http')
var url = require('url')

var route = require('./modules/route.js')

http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type':'image/jpeg'})
  if(req.url !== '/favicon.ico'){
    var pathName = url.parse(req.url).pathname.replace(/\//,'') //http://localhost:8000/login就是login
    console.log(pathName);
    try{  //最简单的异常处理
      route[pathName](req, res)
    } catch(err){
      route['home'](req, res)  //默认路由，如果不存在哪个路由，就返回这个路由
    }

  }
  //res.end()
}).listen(8000)

console.log('Server running at http://localhost:8000');

```

route.js

```javascript
var file = require('./file.js')

module.exports = {
  home: (req, res) => {
    res.write("欢迎来到首页")  // 默认路由
  },
  login: (req, res) => {
    res.write("登录界面")  // 处理路由
  },
  registor: (req, res) => {
    res.write("注册界面")  // 处理路由
  },
  img: (req, res) => {
    //读取文件，也新建一个模块
    file.readImg('./images/1.jpg', res)
  }
}
```

file.js

```javascript
var fs = require('fs')

module.exports = {
  readImg: (file, res) => {
    fs.readFile(file, 'binary', (err, data) => {
      if(err) throw err;
      res.writeHead(200, {'Content-Type':'image/jpeg'})
      res.write(data, 'binary')
      res.end()
    })
  }
}

```



**文字和图片一起显示**

使用html文件

views/home.html

```html
<!<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>首页</title>
  </head>
  <body>
    首页
    <img src="./img" alt="">
  </body>
</html>

```

file.js

```javascript
var fs = require('fs')

 //读html文件
module.exports = {
  readFile: (file, res) => { 
    fs.readFile(file, 'utf-8', (err, data) => {
        if(err) throw err;
          res.writeHead(200, {'Content-Type':'text/html; charset=utf-8'})
          res.write(data)
          res.end()
  })},
  readImg: (file, res) => {
    fs.readFile(file, 'binary', (err, data) => {
      if(err) throw err;
      res.writeHead(200, {'Content-Type':'image/jpeg'})
      res.write(data, 'binary')
      res.end()
    })
  }
}

```



route.js

```javascript
var file = require('./file.js')

module.exports = {
  home: (req, res) => {
    //res.write("欢迎来到首页")  // 默认路由
    file.readFile('./views/home.html', res)

  },
  login: (req, res) => {
    res.write("登录界面")  // 处理路由
  },
  registor: (req, res) => {
    res.write("注册界面")  // 处理路由
  },
  img: (req, res) => {
    //读取文件，也新建一个模块
    file.readImg('./images/1.jpg', res)
  }
}

```

router_03.js

```javascript
var http = require('http')
var url = require('url')

var route = require('./modules/route.js')

http.createServer((req, res) => {
  if(req.url !== '/favicon.ico'){
    var pathName = url.parse(req.url).pathname.replace(/\//,'')
    console.log(pathName);
    try{  //最简单的异常处理
      route[pathName](req, res)
    } catch(err){
      route['home'](req, res)  //默认路由，如果不存在哪个路由，就返回这个路由
    }

  }
  //res.end()
}).listen(8000)

console.log('Server running at http://localhost:8000');

```



###### 路由传递参数

**1.get方式**

login.html

```html
    <form action="./login" method="get">
      <label for="email">
        邮箱: <input type="text" name="email" value=""/>
      </label>
      <label for="password">
        密码: <input type="password" name="password" value=""/>
      </label>
      <label for="submit">
        <input type="submit" value="提交" />
      </label>
```

toute.js

```javascript
var file = require('./file.js')
var url = require('url')

module.exports = {
  login: (req, res) => {
    var urlObject = url.parse(req.url, true).query  // 通过url解析html表单数据
    console.log(urlObject.email)
    console.log(urlObject.password)
    file.readFile('./views/login.html', res, req)
  }
}


```

**2.post方式**

route.js

```javascript

var file = require('./file.js')
var url = require('url')
var queryString = require('queryString')


module.exports = {
  login: (req, res) => {
    //post方式
    var post = ''
    req.on('data', (chunk) => {
      post += chunk
    })
    req.on('end', () => {
      var urlObject = queryString.parse(post) //返回一个url对象
      console.log(urlObject.email);
      console.log(urlObject.password);
    })
    file.readFile('./views/login.html', res, req)
  }
}

```



**输入的内容回显到本页面**

file.js	

```javascript
  postReadFile:  (file, res, req, post) =>{
    var urlObject = queryString.parse(post)
    var array = ['email', 'password']
    var reg;

    fs.readFile(file, 'utf-8', (err, data) => {
        if(err) throw err;
          res.writeHead(200, {'Content-Type':'text/html; charset=utf-8'})
          //处理
          for(var i=0; i<array.length; i++){
            reg = new RegExp('{' + array[i] + '}', 'gi')
            data = data.replace(reg, urlObject[array[i]])
          }
          if(urlObject.email && urlObject.password){
            data = data.replace(new RegExp('{infoClass}', 'gi'), '')
            data = data.replace(new RegExp('{formClass}', 'gi'), 'hide')
          }else{
            data = data.replace(new RegExp('{infoClass}', 'gi'), 'hide')
            data = data.replace(new RegExp('{formClass}', 'gi'), '')
          }
          res.write(data)
          res.end()
        })
},
```

route.js

```javascript
  login: (req, res) => {
    //post方式
    var post = '';
    req.on('data', (chunk) => {
      post += chunk
    })
    req.on('end', () => {
      file.postReadFile('./views/login.html', res, req, post)
    })
  },
```

login.html

```html
    <div class="{infoClass}">
      Email:{email} <br>
      密码:{password}
    </div>
    <form action="./login" method="post" class="formClass">
      <label for="email">
        邮箱: <input type="text" name="email" value=""/>
      </label>
      <label for="password">
        密码: <input type="password" name="password" value=""/>
      </label>
      <label for="submit">
        <input type="submit" value="提交" />
      </label>
```



#### async

`npm install async -D`

常用功能:

```javascript
//1.串行无关联(第一个参数是数组，第二个参数是个回调函数)
//运行时间是两个运行时间总和
console.time('test')
async.series([
  (callback) => { //数组的第一个元素
    setTimeout(()=>{
      callback(null,'one')
    },200)
  },
  (callback)=>{ //数组的第一个元素
    setTimeout(()=>{
      callback(null,'two')
    },5000)
  }
],
(err, results) => {  // 第二个参数
  console.log(results);
  console.timeEnd('test') //看一下执行时间
}
)
```

```javascript
// 2.串行无关联(第二种用法，第一个参数是对象)
console.time('test')
async.series({
  one: (callback) => {
    setTimeout(() => {
      callback(null, '1')
    },2000)
  },
  two: (callback) => {
    setTimeout(() => {
      callback(null, '2')
    ,3000)
  }
  }, (err, results) => {
    console.log(results);
    console.timeEnd('test')
})
```

```javascript
// 3.并行无关联（第一个参数是数组）
//时间取最长的
async.parallel([
  (callback) => {
    setTimeout( ()=>{
      callback(null, 'one')
    },2000)
  },
  (callback) => {
    setTimeout( ()=>{
      callback(null, 'two')
    },5000)
  },
], (err,results)=>{
  console.log(results);
  console.timeEnd('test')   //运行时间5s，取最长的
})
//也可以第一个参数是对象
```

```javascript
// 4.串行有关联
async.waterfall([
  function (callback){
    callback(null,'one','two')
  },
  function (arr1,arr2,callback){
    callback(null, arr1,arr2,'three')
  },
  function (arr1,arr2,arr3,callback){
    callback(null, [arr1,arr2,arr3,'done'])
  }
],function(err,results){
  console.log(results);
})
//这里没加异步，按照串行顺序依次下来的

```







## 六. Socket

<img src="http://image.xpshuai.cn/socket.png"></img>





#### 1). 基于`net`模块实现socket

**聊天案例**

server.js

```javascript
var net = require('net')
var chatServer = net.createServer(),
    clientMap = new Object()

var i = 0; //客户端连接流水号
chatServer.on('connection', function (client){
  client.name = ++i
  clientMap[client.name] = client
  console.log("客户端 "+ client.name +" 连接成功~");

  //对客户端发送消息的监听
  client.on('data', function (data){
    console.log("客户端传来：" + data);
    broadcast(data, client) //广播给其他
  })

  //数据错误的处理
  client.on('error', function (err){
    console.log("client error: " + err);
    client.end()
  })

  //客户端关闭事件
  client.on('close', function (data){
      delete clientMap[client.name]
      console.log(client.name + "下线了");
      broadcast(client.name+"下线了", client) //广播给其他
  })
})

chatServer.listen(9000)  //监听端口

//消息广播
function broadcast (message, client){
  for(var key in clientMap){
    clientMap[key].write(client.name + ' say:' + message + "\n")
  }
}

```

client.js

```javascript
var net = require('net')
var port = 9000
var host = '127.0.0.1'

var client = new net.Socket()
client.setEncoding = 'UTF-8'

//连接服务器
client.connect(port, host, function () {
  client.write("您好")
})

//监听服务端消息
client.on('data', function (data) {
  console.log("服务端传来：" + data);
  say()
})

client.on('error', function (err){
  console.log(err);
})

client.on('close', function (err){
  console.log('connection closed');
})

const readline = require('readline')
const r1 = readline.createInterface({
  input: process.stdin,   //标准输入
  output: process.stdout
})

function say() {
  r1.question('请输入：', (inputStr => {
    if (inputStr != 'bye'){
      client.write(inputStr + '\n')
    }else{
      client.distory() //关闭连接
      r1.close()
    }
  }))
}

```



#### 2). 浏览器原生支持的`WebSocket`

`npm install ws`

案例同上

server.js

```javascript
var wbsocketServer = require('ws').Server,
    wss = new wbsocketServer({port:9001})
var clientMap = new Object()
var i=0

wss.on('connection', function (ws){
  ws.name = ++i
  clientMap[ws.name] = ws
  console.log(ws.name + " 上线了");

  ws.on('message', function (message){
    broadcast(message, ws)
  })
  ws.on('close', function (message){
    // global.gc()  //调用内存回收
    console.log(ws + " 离开");
  })
})


//广播方法
function broadcast(msg, ws){
  for(var key in clientMap){
    clientMap[key].send(ws.name + '说：' + msg)
  }
}

```

client.js

```javascript
var WebSocket = require('ws')
var ws = new WebSocket('ws://127.0.0.1:9001/')

ws.onopen = function () {
  ws.send('大家好')
}

ws.onmessage = function (event) {
  var chatroom = document.getElementById('chatroom')
  chatroom.innerHtml += '</br>' + event.data
}

ws.onclose = function () {
  alert('Closed')
}

ws.onerror = function (err) {
  alert('Error:' + err)
}

```



chat.html

```html
  <body>
  <div id="chatroom"></div>
  <input type="text" name="sayinput" id="sayinput" value="">
  <input type="button" name="send" id="sendbutton" value="发送">

  <script src="client.js"></script>
  <script type="text/javascript">
      function send(){
        ws.send(sayinput.value)
        sayinput.vaule = ""
        }
        document.onkeyup = function(event){
          if (event.keycode == 13){
            send()
          }
        }
        sendbutton.onclick = function (){
          send()
        }

  </script>
  </body>
```



#### 3). `socket.io`实现socket

当浏览器不支持html5时，就不能用websocket了

借助express实现: `npm i express socket.io`

socket_io.js

```javascript
var app = require('express')()
var http = require('http').Server(app)
vat io = require('socket.io')(http)
var fs = require('fs')

app.get('/', (req,res)=>{
  function callback(data) {
    res.send(data.toString())
  }
  fs.readFile('socketIOClient.html', function (err, data){
      if(err){
        console.log(err);
        console.log("文件不存在");
      }else{
        callback(data)
      }
  })
})

// app.get('/socket.io.js')


//socket.io的设置
//在线用户
var onlineUsers = {}
//在线人数
var onlineCount = 0

var 1 = 0

io.on('connection', function (socket){
  console.log("有人上线了");
  //监听新用户加入
  socket.name = i
  onlineUsers[socket.name] = socket

  //监听用户退出
  socket.on('disconnect', function (){
    console.log("有人退出了");
    delete onlineUsers[socket.name]
  })

  //监听用户发布的聊天内容
  socket.on('message', function (msg){
      broadcast(msg, socket)
  })
})

//广播方法
function broadcast(msg, socket){
  for(var key in onlineUsers){
    onlineUsers[key].send(socket.name + '说：' + msg)
  }
}

http.listen(9002, function (){
  console.log("");
})

```

html

```javascript
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
    <style>
    #chatroom{
      width:400px;
      height:600px;
      overflow:auto;
      border:1px solid bule
    }
  </style>
  <script src="./socket.io.js"></script>
  </head>
  <body>
      <script type="text/javascript">
        var iosocket = null;
        windows.onload = function (){
          iosocket = io.connect('http://127.0.0.1/9002')

          iosocket.on('connect'. function (){
            iosocket.send('hello, from inclient')
          })

          //接收数据
          iosocket.on('message', function (message){
            var chatroom = document.getElementById("chatroom");
            chatroom.innerHtml += message = '</br>'
          })

          //断开连接
          iosocket.on('disconnect', function (){
            console.log("服务端关闭");
          })

          function send(){
            iosocket.send(sayinput.value)
            sayinput.value = "''"
          }
          document.onkeyup = function(event){
            if (event.keycode == 13){
              send()
            }
          }
          sendbutton.onclick = function (){
            send()
          }
        }
      </script>


  <div id="chatroom"></div>
  <input type="text" name="sayinput" id="sayinput" value="">
  <input type="button" name="send" id="sendbutton" value="发送">


  </body>
</html>

```



## 七.MongoDB

#### 安装(Linux下)

**1. 通过公钥对资源包一致性和真实性进行验证：**

```bash

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4

```

**2. 运行下行命令，创建文件/etc/apt/sources.list.d/mongodb-org-4.0.list：**

```bash
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
```

**3. 在安装之前，先更新系统资源包：**

```bash
sudo apt-get update
```

**4. 安装mongodb资源包：**

```bash
# 安装MongoDB server
sudo apt-get install -y mongodb-org 

# 安装MongoDB 客户端
sudo apt-get install mongodb-clients
```

**4. 检验是否安装成功：**

启动mongod服务：

```bash
sudo service mongod start 

#查看mongo版本:
mongo --version
```

**5.登录MongoDB并设置密码**

```bash
# 登录
mongo
# 切换到admin数据库
use admin
# 创建admin账号，密码::admin111 权限：root权限，授权的数据库名：admin
db.createUser({user:"admin", pwd:"admin111", roles:[{role:"root", db:"admin"}]})
```

编辑MongoDB配置文件

```bash
vim /etc/mongod.conf

net:
  port: 27017  # 端口
  bindIp: 0.0.0.0  # 允许访问的地址，0.0.0.0表示所有

security:
  authorization: enabled
```

重启MongoDB服务

```bash
systemctl restart mongod.service
```

测试登录

```bash
mongo -u admin -p admin111 --authenticationDatabase admin
```



##### 学习

> 与mysql语法对比：https://www.jb51.net/article/65626.htm?tdsourcetag=s_pctim_aiomsg

###### 术语、概念



![](http://image.xpshuai.cn/socket.png)

![](http://image.xpshuai.cn/mysql2.png)

**MongoDB集合**

一个数据库可以有多个集合(对应mysql的table)

**MongoDB文档**

键值对

如：`{'name':'xps', 'like':['code','play']}`



**MongoDB数据类型**

![](http://image.xpshuai.cn/type.png)



#### 常用命令

###### 1.数据库操作

查看帮助

```bash
help
db.help()
db.test.help()
df.test.find().help()
```

创建/切换数据库

```bash
use music  # 如果不存在就创建，存在且切换
```

查询数据库

```bash
show dbs   # 当数据库为空时，是不显示的

# 我们向一个集合里面插入一个文档
db.user.insertOne({'name':'xps','like':['code','play']})
# 查看集合里面的文档(行)
db.user.find()
```

查看当前使用的数据库

```bash
db
#
db.getName()
```

显示当前DB状态

```bash
db.stats()
```

查看当前DB版本

```bash
db.version()
```

查看当前DB的链接机器地址

```bash
db.getMongo()		
```

删除数据库

```bash
db.dropDatabase()
```



###### 2.集合（Collection）操作

....写了半天，忘了保存，算了，放图片吧....

<img src="http://image.xpshuai.cn/collec.png"></img>



###### 3.文档操作

对集合内数据增删改查



<img src="http://image.xpshuai.cn/collec.png"></img>



删除

```bash
db.user.remove({'name':'xps'})
db.user.delete({'name':'xps'})
db.user.deleteMany({'name':'xps'})
```





<img src="http://image.xpshuai.cn/q1.png"></img>

<img src="http://image.xpshuai.cn/q2.png"></img>

<img src="http://image.xpshuai.cn/q3.png"></img>





## 八. Express框架

> 一个node.js的web框架



#### 1). 准备

**安装**

`npm install express --save`, Windows下还有:`npm install -g express-generator `

`express -h` 测试安装成功

新建项目：`express test_express    ` 

**目录结构：**

package.js是需要的依赖，切换到项目根目录手动安装`npm install`

app.js是入口

www/bin下面是端口等配置



**启动项目：**

```bash
npm start
```



#### 2). Express初始化项目详解

一些模块

```json
  "dependencies": {
    //"body-parser": "~1.15.2",   // 解析前端提交的数据
    "cookie-parser": "~1.4.4",   //解析cookie
    "debug": "~2.6.9",    // 用来显示调试信息
    //"ejs": "~2.5.2",   //模板，渲染页面   
    "express": "~4.16.1",  
    "http-errors": "~1.6.3", 
    "jade": "~1.11.0",   //模板，渲染页面
    "morgan": "~1.9.1"   //日志，控制后格式化显示信息
  }
```



`bin/www`入口文件

实现了创建服务和中间件、路由配置

```javascript

normalizePort() 方法，对端口进行简单处理


1.路由中间件
2.自定义中间件（app.set('port', port)这样）

app.use()

```



`app.js`

```javascript
app.use(express.static(path.join(__dirname, 'public')));   //设置静态文件路径
...
```

```javascript
var usersRouter = require('./routes/users');
app.use('/users', usersRouter);
```



`router/xxx.js` 路由的中间件

```
 //这里的index就是view下面的index.jade
res.render('index', { title: 'Express' });  //渲染模板 ， index不用写后缀名
```

```javascript
#比如 index.js
var express = require('express');
var router = express.Router();

// get('/', xxx) 因为前面写了/user，所以这里只写/即可
router.get('/', function(req, res, next) { 
  res.send('respond with a resource');
});

module.exports = router;  // 暴露出去，才能以中间件的形式存在
```



`views/xxx.jade` 模板文件



#### 3). Express中的路由

users.js

```javascript
var express = require('express');
var router = express.Router();

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});

// 第二层路由，第一层在app.js里
router.get('/list', function(req, res) {
  res.send('user list');
});

// 可以写成表达式（正则）
router.get('/a*d', function(req, res) {
  res.send('表达式路由');
});


//转到一个文件
router.get('/form', function(req, res) {
  res.sendFile(__dirname + '/form.html')
});

//post请求
router.post('/save', function(req, res) {
  res.send("表单提交!")
});

// all， post和get都接收
router.all('/all',function(req, res) {
  res.send("post和get都接收!")
});


module.exports = router;

```

form.html

```html
<form action="/users/save" method="post" class="formClass">
    <label for="username">
        username: <input type="text" name="username" value=""/>
    </label>
    <label for="submit">
        <input type="submit" value="提交" />
    </label>

</form>
```



#### 4). Express中的模板

ejs/jade等

ejs比较简单

```javascript
#安装（express内置了ejs）
npm install ejs --save

#配置ejs模板引擎
app.set('view engine','ejs')

#渲染页面
render(filename,json)接收两个参数。

#如下
app.get('/learnejs',function(req,res){
   let arr=['吃饭','睡觉','打豆豆']
    res.render('index',{
        list_a:arr
    })
})


#index.ejs
<ul>
    <%for(var i=0;i<list_a.length;i++){%>
    <li><%=list[i]%></li>
    <%}%>
</ul>
```

渲染页面

```javascript
#
render(filename,json)接收两个参数。
```

常用标签

```javascript
<%  %>   流程控制标签
<%= %>	 输出标签（原文输出html标签）
<%- %>	 输出标签（html会被浏览器解析）
<%# %>	 注释标签
%		 对标记进行转义
```

`includes`,在一个模板里面引用另一个模板

```javascript
<%- include("模板路径", {user:"传递的参数内容"})  %>
```







## 九.mocha

> 前端测试框架

浏览器和Node环境都可使用



安装：

```

# 安装
npm install mocha -g

#命令
mocha

#本地安装
npm install mocha -D

# 修改配置文件中test命令
"test": "......./bin/mocha"


# 运行
npm test

```



#### 基本用法

**定义测试：**

创建一个js文件

```javascript
describe("Demo", function (){
    describe("方法1", function (){
        before(function (){  //测试之前做的内容
            console.log("---测试之前做的内容")
        })
        after(function (){  //测试之后做的内容
            console.log("---测试之前后做的内容")
        })
       beforeEach(function (){  //每条测试之前做的内容
            console.log("---每条测试之前做的内容")
        })
        context("情景1", function (){
            it("测试1", function (){
                
            })
            it("测试2", function (){
                
            })
        })
    })
})

```

`mocha` 来启动测试/ 或使用本地安装的mocha来测试`npm test`





#### assert断言

使用库：`chai`

- chai: 断言库
- chai: should分割的断言
- chai：expect风格的断言

安装到本地：`npm install  chai -D`



**使用**

```javascript
const chai = require('chai')
const assert = chai.assert

describe('Demo', function (){
  it("使用 assert风格的断言测试", function (){
    var val = 'hello'
    assert.typeOf(val, 'string')
    assert.equal('hello')
  })
})

```

```javascript
const chai = require('chai')
const sholud = chai.sholud

describe('Demo', () => {
  it("使用 sholud风格的断言测试", function (){
    var val = 'hello'
    val.sholud.exist
    val.sholud.be.a('string')
    val.sholud.equal('hello')
    val.sholud.not.equal('hello')
    val.sholud.have.length()
    //多个条件逻辑与
    val.sholud.equal('hello').and.exist.and.be.a('string')
  })
})
```

```javascript
const chai = require('chai')
const expect = chai.expect

describe('Demo', () => {
  it("使用 expect风格的断言测试", function (){
    var val = 'hello'
    var number = 3
    
    expext(number).to.be.at.most(5)
    expext(number).to.be.at.least(3)
    expext(number).to.be.at.within(1,8)
      
    expect(value).to.exist
    expect(value).to.be.a('string')
    expect(value).to.equal('hello')
    expect(value).to.not.equal('hello')
    expect(value).to.have.length(5)
  })
})
```





#### 运行多个测试

```bash

mocha --recursive   # 递归
```



好了，再学就跑偏了...... :smile:





































---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/node-js-%E5%85%A5%E9%97%A8/  

