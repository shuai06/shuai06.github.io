# ubuntu server-帮助&输出与重定向




#### 获取帮助

###### man

```bash
man cmd

man -k keyword    #搜索关键词（不知道具体命令）
man -k 'list directory contents'   # 多个关键字

# 每个命令的 手册页 可以使用数字编号引用片段（section）
man 5 passwd   #（ 按section查看）
- 1：  用户命令
- 2：  系统调用
- 3：  高级Unix编程库文档（程序员常用）
- 4：  设备借口和驱动信息（很少使用）
- 5：  文件描述（系统配置文件）
- 6：  Games
- 7：  文件格式，惯例，编码(ASCII等)
- 8：  系统命令和服务器
```



###### info

GUN不太喜欢慢，因此开发了`info`
有时候优于man，有时不是
`info ls`

`/usr/share/doc` # 部分软件包没有列入man或者info的文档

###### cmd --help / -h

绝大部分的不成文规定，并不都是，比如`ls -h`

#### Shell输入与输出

###### 输出重定向

```shell
ls a/ > file   # 会覆盖

ls a >> file    # 追加

head /proc/cpuinfo 

head  < /proc/cpuinfo    # 输入重定向


#
`set -C` 之后不得覆盖
```

###### 管道

```bash
head /proc/cpuinfo | tar a-z A-Z

ifconfig | grep inet | awk'{print $2}' | awk -F. '{print $2}'   # -F 指定分隔符，这里以.分割
```

###### 标准错误 stderr(报错包含重要信息)

```bash
# 写脚本的时候有用，可以把错误信息报错到某个文件
ls /abc 2>e   # 2:标准错误      1:标准输出          0:标准输入
ls 0< a/
ls 1> file1
ls b 2> err1


ls a/ > f1 2>&1   # 标准输出、标准错误都输出到文件f1  （&1代表标准输出）
```

###### 常见报错信息

<img src="https://image.geoer.cn//error.png"></img>









---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E8%BE%93%E5%87%BA%E4%B8%8E%E9%87%8D%E5%AE%9A%E5%90%91/  

