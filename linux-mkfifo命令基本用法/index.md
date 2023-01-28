# Linux mkfifo命令基本用法




### 先复习下linux命令执行顺序

```bash
# 通常，终端只能执行一条命令, 如果要执行多条命令
# 顺序执行多条命令，可以用分号;
cmd1;cmd2;cmd3

# 条件执行多条命令，使用&&(前一个命令执行成功，即$?=0时，执行下一条命令，否则不执行)和||(前一个命令执行失败，既$?≠0时，执行下一条命令)
cmd1&&cmd2||cmd3

#$?:上一次命令的返回结果 --->  为0代表执行成功，不为0则为执行失败
```





### 管道命令(pipe)

- 管道是一种`通信机制`，用于进程间的通信（也可通过socket进行网络通信），表现出来的形式将前面的每一个进程的输出，直接作为下一个进程的输入
- <font color=red>管道命令仅能处理stdout，而error则会忽略</font>





### 常见管道命令

- cut、grep、sort、wc、uniq
- tee：重定向，既能在屏幕输出，又能保存到文件中
- tr、col、join、paste、expand、split





### 有名管道+无名管道=管道

- 有名管道(FIFO文件)：就是 有文件名的管道, 可以用于任意两个进程间的通信
- 无名管道(pipe文件)：就是没有文件名的管道, 只能用于父子进程之间的通信



### mkfifo

> mkfifo可以创建命名管道



```bash
# |（竖线）为管道，是两个进程之间的通信通道
例如：ls|grep txt
#ls和grep由|分开，管道创建了程序之间的通信通道，将ls的输出作为输入传给grep


# 由mkfifo创建出来的就是一个命名管道
例如：mkfifo pipe2
#pipe2就是一个命名管道。
```



**命名管道的作用**

```bash
# 可以将输出信道化到不同终端、
例如：
在第一个终端执行
ls > pipe2
在第二个终端执行
cat < pipe2（或cat pipe2,是取一次。cat < pipe2是持续输入，只要有内容传到pipe2中，就会有内容输出）

#pipe2更像是一个临时存储的地方，使用cat pipe2取过内容之后，再执行cat pipe2 ，则不会有显示
```



命名管道可以像正常文件一样访问，在文档类型可以看到为p：

```bash
ubuntu@VM-0-14-ubuntu:~$ ll pipe2
prw-rw-r-- 1 ubuntu ubuntu 0 Jun  7 09:35 pipe2|
```

删除就像正常文件一样使用rm删除即可





> 可以使用指定的名称创建先进先出文件（FIFO）

 

**语法格式：**mkfifo [参数] [名称]



**常用参数：**

| m <模式> | 设置权限模式，类似chmod                  |
| -------- | ---------------------------------------- |
| -Z <CTX> | 将每个创建的目录SELinux安全环境设置为CTX |



**实例：**

创建FIFO文件 /tmp/fifo：

```bash
ubuntu@VM-0-14-ubuntu:~$ mkfifo /tmp/fifo

ubuntu@VM-0-14-ubuntu:~$ ls -l /tmp/fifo 
prw-rw-r-- 1 ubuntu ubuntu 0 Jun  7 09:47 /tmp/fifo
```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/linux-mkfifo%E5%91%BD%E4%BB%A4%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95/  

