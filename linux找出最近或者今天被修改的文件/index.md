# linux找出最近或者今天被修改的文件



  
参考：http://www.hackdig.com/01/hack-42547.htm


  
#### 0x01 列出某个目录下今天创建或者修改的文件

###### 1 显示目录home/ym下，今天创建或者修改的文件

`ls  -al --time-style=+%D | grep 'date +%D'`

参数解释：

```
-a - 列出所有文件，包括隐藏文件
-l - 启用长列表格式
--time-style=FORMAT - 显示指定 FORMAT 的时间
+％D - 以 ％m/％d/％y （月/日/年）格式显示或使用日期-a - 列出所有文件，包括隐藏文件
-l - 启用长列表格式
--time-style=FORMAT - 显示指定 FORMAT 的时间
+％D - 以 ％m/％d/％y （月/日/年）格式显示或使用日期
```



###### 2 按字母顺序对结果排序显示

`ls -alX --time-style=+%D | grep 'date +%D'`

###### 3 按文件大小从大到小对结果排序显示

`ls -alS --time-style=+%D | grep 'date +%D'`



#### 0x02 列出某天所有被修改文件

###### 1 列出当前目录今天被修改的文件

`find . -maxdepth 1 -newermt "2017-1-8"`

`find . -maxdepth 1 -newermt "1/8/2017"`

###### 2 列出系统中今天被修改的所有文件

`find .  -newermt "2017-1-8"`

`find . -newermt "1/8/2017"`



#### 0x03  查找被`访问`过的文件

###### 1 今天被访问的文件

` find /home -atime 0 #查看home 目录下今天被访问的文件`

###### 2 查看几天之内被访问的文件

`find . -atime +2  # -atime n,  File  was last accessed n*24 hours ago.；查看当前目录三天之内被访问的文件`



#### 0x04 查看被`修改`过的文件

###### 1 今天被访问的文件

` find /home -ctime 0  #查看home 目录下今天被修改过的文件`

###### 2 查看几天之内被访问的文件

`find . -ctime +2  # -ctime n,  File  was last changed n*24 hours ago.；查看当前目录三天之内被修改过的文件`



#### 0x05 更多查看文件的用法

... ...

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/linux%E6%89%BE%E5%87%BA%E6%9C%80%E8%BF%91%E6%88%96%E8%80%85%E4%BB%8A%E5%A4%A9%E8%A2%AB%E4%BF%AE%E6%94%B9%E7%9A%84%E6%96%87%E4%BB%B6/  

