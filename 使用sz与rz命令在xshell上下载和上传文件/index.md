# 使用sz与rz命令在Xshell上下载和上传文件


### 概述

#### 安装

```bash
sudo apt-get install lrzsz
```



#### 常见参数

```bash
参数	参数说明
-a	以文本方式传输（ascii）
-b	以二进制方式传输（binary）
-e	对控制字符转义（escape），这可以保证文件传输正确
```



#### 两个命令：

- sz：从xshell里面的服务器下载文件到本机
- rz：从主机上传文件到xshell里面的服务器

### sz用法：

 下载一个文件

```bash
sz file_name 
```

下载多个文件

```bash
sz file_name1 file_name2
```

下载指定目录下的所有文件，不包含该目录下的文件夹

```bash
sz test_dir/*
```



### rz用法：

输入rz回车后，会出现文件选择对话框，选择需要上传文件，一次可以指定多个文件，上传到服务器的路径为当前执行rz命令的目录。



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E4%BD%BF%E7%94%A8sz%E4%B8%8Erz%E5%91%BD%E4%BB%A4%E5%9C%A8xshell%E4%B8%8A%E4%B8%8B%E8%BD%BD%E5%92%8C%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6/  

