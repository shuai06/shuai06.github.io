# git commit 提交规范


## 简介

commit message 应该清晰明了，说明本次提交的目的。而且多人协作的时候，有问题也方便查看提交日志

目前使用最广的写法是Angular 规范(https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0)，它比较合理和系统化，并且有配套的工具



## Commit message 作用

格式化的Commit message，有几个好处。

**（1）提供更多的历史信息，方便快速浏览。**

比如，显示每次发布后的变动，每个commit占据一行。你只看行首，就知道某次 commit 的目的。



**（2）可以过滤某些commit（比如文档改动），便于快速查找信息。**

比如，下面的命令仅仅显示本次发布新增加的功能。

```bash
git log <last release> HEAD --grep feature
```



**（3）可以直接从commit生成Change log。**

Change Log 是发布新版本时，用来说明与上一个版本差异的文档。



## Commit message 的格式规范

每次提交，Commit message 都包括三个部分：Header，Body 和 Footer。

```bash
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>

```

其中，Header 是必需的，Body 和 Footer 可以省略。

一、Head：（一般简单项目只写head也够）

**格式**：`type <scope> : subject`

**示例：**`feat(订单模块)：订单详情接口增加订单号字段`

其中， feat对应type字段；订单模块对应scope(若果scope有内容，括号就存在)；“订单详情接口增加订单号字段”对应subject，简要说明此次代码变更的主要内容。



**1、type**（必须） 必须要填写，这是表明此次commit的类型，具体有一下几种类型：

```
feat：新功能
fix：修复bug
docs：文档改变
style：代码格式改变
refactor：重构某个功能
perf：性能优化
test：增加测试
build：构建项目，如替换了构建工具
revert：撤销上次的commit
chore：构建过程或者辅助工具的变动
```

如果`type`为`feat`和`fix`，则该 commit 将肯定出现在 Change log 之中。其他情况（`docs`、`chore`、`style`、`refactor`、`test`）由你决定，要不要放入 Change log，建议是不要。

**2、scope**（可选）用于说明此次commit提交作用的范围，例如视图层，控制层，数据之久层，项目配置等等

**3、subject**（必须）主题必须填写，用于描述此次commit的内容，简单描述即可，例如如何重构某一项功能。

```bash
# subject
1.以动词开头，使用第一人称现在时，比如change，而不是changed或changes

2.第一个字母小写

3.结尾不加句号（.）
```









二、Body

Body 部分是对本次 commit 的详细描述，可以分成多行，每行尽量不超过72个字符





三、Footer

Footer 部分只用于两种情况。

1.不兼容变动

2.关闭Issue







## Pycharm的Git Commit Template插件

![下载](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/2022-05-10-141402.png)



![commit](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/2022-05-10-141430.png)



![点击](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/2022-05-10-141459.png)



![填写](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/2022-05-10-141528.png)



## 生成Change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成

...用的时候再来了解

























---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/git-commit-%E6%8F%90%E4%BA%A4%E8%A7%84%E8%8C%83/  

