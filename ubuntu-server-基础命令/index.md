# Ubuntu server-基础命令

  
#  基本Shell常识

```shell
ctrl + D   # 结束当前输入
ctrl + c   # 强制结束一切

#
bash
```





# ls

###### 参数:

```
-l
-a
-d   只显示目录自身信息
-i   显示inode信息
-S   按文件大小排序
-r   倒序
-t   按修改时间排序
-h   文件大小便于人类阅读方式
```









# cp

```shell
cp file1 file2

cp file1 f2 dir/
```



###### 参数:

```shell
-R/r  拷贝目录
-l  硬链接拷贝(ls -li)   # node的值是相同的,对应的硬盘数据块是相同的,只是多了一个指向
-s 软连接拷贝 (ls -li查看时候第一个字母是'l')            #inode值相同, 只是指向了真实的文件,快捷方式
-S 目标名称添加后缀   `cp -S -bak b b1`  # 把b拷贝为b1的时候,目标文件b1加上后缀-bak
-u 源比目标新时才拷贝   `cp -u aa  /home/a` # 保存更加新的文件


```







# mv

###### 参数:

```bash
mv f1 f2
mv f1 f2 dir/
-f # 强制移动
```





# touch

###### 参数:

```bash
- 如果文件名存在,修改文件mtime, 但不修改内容

```







# rm

###### 参数:

```bash
-i   # 每删除之前提醒
-d   # 删除空目录

rm -rf
```





# echo   

将命令参数显示在stdout

###### 参数:

```bash
-n   # 显示结束不换行
-e   # 解释反斜线转义符
      `echo \"n"`
      `echo -ne 123\\b`
      echo -e a\\n

```



# 目录结构

目录  > 分区

```
/bin/
/boot/
/dev/
/etc/
/home/
/lib/
/media/
/mnt/
/opt/
/sbin/
/srv/
/temp/
/usr/
/var/
/root/
/proc/
```





# mkdir

###### 参数:

```bash
-p # 按需创建父目录
```









# pwd

###### 参数:

```bash
-P     # 物理路径(软连接对应的真实路径)
-L     # 逻辑路径(软连接自身路径)
```





# rmdir

###### 参数:

```bash
-p   # 递归删除
```

###### 通配符:

```

```







# grep

`sudo grep root /etc/*`



###### 参数:

```bash
-i     # 忽略大小写
-v     # 反相匹配
-n     # 显示行号
-r     # 递归目录和子目录中所有文件
-c     # 显示目标文件中包含关键字的行数
-f  1.txt 2.txt    # 1.txt中多个关键字同时匹配    
-E  '1|2|3' a.txt  # 或者1或者2或者3  
grep a[123] a.txt   a1,a2,a3
```







# less

more的增强版

###### 参数:

```bash

```

###### 快捷键:

```bash
z/b   # 向前/后翻页
v     # 进入编辑模式
g/G   # 直接跳过第一行/最后一行(或第n行)
/word    # 向前搜索关键词
?word    # 向后搜索关键词
n/N       # 正向/反向继续搜索关键字
q      # 退出

grep xxx/words | less
```





# nano

###### 参数:

```bash

```





# head  / tail

显示文件头/尾部内容, 默认10行

###### 参数:

```bash
-n   # 指定显示的行数   比如: -3
-f     #  实时显示    tail -f a.txt
		# tailf 命令
```







# diff

###### 参数:

```bash
-u  # 统一格式输出
-y  # 并排输出比较
	# | "不同", ; 配合-w参数限制宽度
-w   # 忽略空格
-i   # 忽略大小写
```





# file

检测文件格式

顺序执行三种测试集:

- filesystem    匹配系统头文件<sys/stat.h>
- magic         , -l 查看
- language, 匹配文件起始位置的字符类型
- 一种测试匹配即停止检测,全都不匹配返回data

###### 参数:

```bash
-f   # 文件列表
-ib  # mime类型
```









# find

###### 参数:

```bash
-name
-type   # b  c  d  f  l
-user
-mtime +1 -mtime -20
   关于时间的概念:
   atime, ctime
   amin, nmin, cmin
-cnewer file   # 比这个文件更加新的文件
   
   find / -mtime 1  # 昨天到前天,发生修改的文件
   find / -mtime +1  # 昨天之前所有修改的,,,
   find / -mtime -1  #  昨天到now,发生修改的文件
```





# stat

查看文件详细信息

###### 参数:

```bash

```





# locate

基于文件索引进行搜索

- 不验证文件是否存在,速度快但结果不准确
- `updatedb`更新索引

###### 参数:

```bash

```







# sort

sort a.txt

###### 参数:

```bash
-r   # 反向排序
-n	 # 按照数值大小排序
-M   # 按照月份排序

# 另一个排序的命令
ls -l --sort=key
   size
   time
   extension
```

**只有先排序,才能取唯一值**



​    





























---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E5%9F%BA%E7%A1%80%E5%91%BD%E4%BB%A4/  

