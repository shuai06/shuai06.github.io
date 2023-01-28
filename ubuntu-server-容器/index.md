# ubuntu server-容器




# 容器
>有了新武器，并不表示旧武器没有用

**题外话**

<img src="http://image.xpshuai.cn/20200510223525320_174889579.png" />

**容器**

<img src="http://image.xpshuai.cn/20200510225525049_1760370298.png" />

SDN -> 软件定义网络

容器 -> 基于操作系统内核，共享宿主机的CPU
        -> 操作系统的进程之间进行隔离
        -> 安全性比虚拟机差一些
         -> 可移植性

**什么是容器**
有个感性的认识即可，现在还没有明确的统一定义，根据不同产品具体理解

<img src="http://image.xpshuai.cn/20200510230949837_2077847896.png" />

<img src="http://image.xpshuai.cn/20200510231620776_117992917.png" />

<img src="http://image.xpshuai.cn/20200511102236109_1751941655.png" />

<img src="http://image.xpshuai.cn/20200511103008352_761812684.png" />

## 常见的容器技术
**Docker vs LXD/LXC**

<img src="http://image.xpshuai.cn/20200511103208058_29787066.png" />

<img src="http://image.xpshuai.cn/20200511103734465_306114851.png" />

不同容器的程序的pid可以相同

<img src="http://image.xpshuai.cn/20200511104628559_66294885.png" />

越狱





## Docker
> 应用程序方面的容器



<img src="http://image.xpshuai.cn/20200511104948809_1439233734.png" />


Docker引擎只可以直接运行在Linux系统上
```bash
利用'Boot2Docker'等适配器帮助，使用轻量级'Linux vm'可在mac和微软系统上运行
```

### 安装Docker引擎(两种方法)
```bash
uname -m    # 仅64位系统保障(但32位系统可用)

##### 第一种办法：直接从ubuntu软件包安装(比较旧), 已经有一个老版本ubuntu包名为docker，
sudo apt install docker.io
systemctl status docker

# 将当前用户加入docker组(不用sudo了)
sudo usermod -aG docker ${USER}
# 重启





##### 第二种办法：
# 安装依赖包
udo apt-get install apt-transport-https ca-certificates  curl software-properties-common
# 添加docker官方GPG key(防止伪造,供应链攻击)
url -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 添加docker官方库更新源
sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs) stable"
# 安装
sudo apt update
sudo apt-get install docker-ce  # 社区版本
```

客户端工具: `docker`


### 基本命令
```bash
# 查看信息
docker version
docker info   # 查看运行了几个容器
# Registry: 应用程序的仓库(基本linux映像，)


# 搜索镜像(有官方有个人的: 用户名/容器名) --> 建议下载官方的image然后自己打造容器
docker search apache2

# 下载镜像
docker pull hello-world
# 下载时候指定-a（会下载所有版本）
docker pull hello-world -a

# 查看镜像
docker images 

# 运行容器
docker run busybox echo "hello"

# 删除image
docker rmi busybox:1-musl

```


### Docker基本概念
**Docker 映像**

- 组成应用程序运行所需的全部文件合集
- 只读/读写 `分层`结构(每次对映像的修改都通过commit形成一个新层)
- `容器层` 是基于映像`可读写`的顶层
- `映像层`所有的Docker映像都源自于一个`基础镜像`(Debian/Ubuntu)  --> 只读，不可修改i
- 其他的功能模块附加于基础影像形成最终程序



<img src="http://image.xpshuai.cn/20200511111809612_1403827030.png" />

- Docker images 是构建Docker container的基础

- Docker 层/容器层

  <img src="http://image.xpshuai.cn/20200511112309748_1071300544.png" />

每个映像有唯一的ID(SHA256/  取前6字节来显示)
```bash
docker images
# 可看到唯一的image id
# TAG:  默认是最新版本(latest)   

docker images --no-trunc
# 可以看到完整的SHA256完整的字节
```



### **Docker容器**

images是只读的，在基础上创建并运行容器(可读写)

```bash
# 查看运行的容器（每个容器都有唯一的id）
docker ps    # 不显示关闭的容器
docker ps -a  # 显示所有状态的容器
#显示字段： 有容器id, 基于image的名字，command, 容器创建时间， status, ports, names(随机生成的,可以改)


# 基于image运行容器(每次运行一次都产生一个新的容器!)
docker run busybox echo "hello"
docker run busybox:1-uclibc echo "hello"  # 指定标签来用哪个(如果本机没有，会自动pull)

# 
docker run -i -t busybox:ubuntu-16.04 


# 删除容器
docker stop d751d48a6d0a  # 如果运行了先stop掉
docker rm d751d48a6d0a

```


pull的时候，默认会去一个人url地址寻找镜像

Docker Resigtry相当于目录，Repository相当于具体文件位置

可以自建映像库

<img src="http://image.xpshuai.cn/20200511143726490_328609255.png" />





### Docker使用
基本命令
```bash
# 运行镜像后交互登录入容器（这里基于ubuntu）
# --name  修改容器的默认名字，指定
docker run --name xxx01 -it ubuntu /bin/bash
# 退出
exit /   ctrl + D

# 退出容器终端，但退出不关闭容器
ctrl + P , ctrl + Q   # 组合

# 启动容器
docker start id_id_id

# 继续进入正在运行的容器
docker attach xxx01

# 重命名容器
docker rename 453dsdaojp test01

# 容器与镜像对比变化
docker diff 453dsdaojp test02
- C:修改了
- A:增加了
- D:删除了



### 日常管理命令
docker start 453dsdaojp
docker restart 453dsdaojp
docker stop 453dsdaojp
docker rename 453dsdaojp
docker pause 453dsdaojp    # 挂起
docker unpause 453dsdaojp  # 恢复
```


创建映像
```bash
docker run -it ubuntu /bin/bash    
....    # 进入容器进行一些了修改
ctrl + p, ctrl + q
docker commit test01 xps/test:0.1  # 创建了新的image
# 然后可以上传到docker Resigtry
# 后面就跟别的image一样使用可以创建容器等
```

后台运行容器
```bash
# 不进入交互方式的shell
docker run -d ubuntu /bin/bash -c "while true; sleep 3; do date";done  
```

查看日志
```bash
docker logs 453dsdaojp
```

**实例**

```bash
# 配置一个apache，并可以外部访问
docker run -dit -p 8080:80 ubuntu /bin/bash  # 端口映射(外部:容器端口)
docker attach doasdkop463fd    # 进入容器
apt update && apt install apache2  # 正常操作
/etc/init.d/apache2 start
ctrl+p, ctrl+Q
# 宿主机浏览器访问 http://宿主机ip:8080


## 服务自动启动(或者手动把下面内容输入进去)
echo '/etc/init.d/apache2 start' >> /etc/bash/rc

## 基于配置好的容器创建新的镜像
docker commit ubuntu xps/apache1:0.1  # 创建了新的image
# 打包好之后上传到hub，就可以正常下载和使用image了
```




### Dockerfiles
>前面的下载，制作，上传，再下载的方法也挺麻烦的

使用Dockerfile自动创建映像（Dockerfile文本文件，包含创建容器的指令）

这里还是以apacha2为例
1.先写配置文件

```bash
# vim Dockerfile
FROM ubuntu     # 从官方下载镜像
MAINTAINER xps <xps@admin.com>
RUN apt update; apt dist-upgrade -y
RUN apt install -y apache2 vim
RUN echo '/etc/init.d/apache2 start' >> /etc/bash/rc
```
2.基于dockerfile来创建容器

```bash
# 标签自己定义即可, 最后的点(.) 表示当前目录下的dockerfile
docker build -t test/apache-server:1.0 .

# 然后run创建启动容器
docker run -dit -p 8080:80 test/apache-server:1.0 /bin/bash
```

>PS: 更深入的Dockerfile知识再另外学







## docker-compose

**作用：**之前运行一个镜像，需要添加大量的参数，可以通过Docker-Compose编写这些参数，Docker-Compose可以帮助我们批量的管理容器，只需要通过一个docker-compose.yml文件去维护。

下载：https://github.com/docker/compose/releases

https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-darwin-aarch64





> 参考文章：https://blog.csdn.net/chenfeidi1/article/details/80866841?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-1-80866841-blog-122964874.pc_relevant_without_ctrlist_v3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-1-80866841-blog-122964874.pc_relevant_without_ctrlist_v3&utm_relevant_index=2
>
>  
>
> https://www.jb51.net/article/237876.htm







### 官方docker-compose.yml

```bash
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:

```





### **Docker-compose管理Mysql和apache容器**

yml文件以key：value方式来指定配置信息，多个配置信息以换行+缩进的方式来区分，在docker-compose.yml文件中，不要使用制表符就是tab键，规则用两个空格缩进

```yml
version: '3.1'   # 指定了 compose file 的版本
services:
  mysql:                                                      #服务名称
    restart: always                                           #代表只要docker启动，那么这个容器就跟着一起启动
    image: daocloud.io/library/mysql:5.7.4                    #指定镜像路径
    container_name: mysql                                     #指定容器名称
    ports:
      - 3306:3306                                             #指定端口的映射
    environment:
      MYSQL_ROOT_PASSWORD: 123456                             #指定MySQL账号root的密码
      TZ: Asia/shanghai                                       #指定时区
    volumes:
      - /opt/docker_mysql_tomcat/mysql:/var/lib/mysql                #指定映射数据卷
  httpd:
    restart: always
    image: php:7.2-apache
    container_name: apache
    ports:
      - 80:80
    environment:
      TZ: Asia/shanghai
    volumes:
      - /opt/docker_mysql_tomcat/www:/var/www/html
      - /opt/docker_mysql_tomcat/logs:/var/log/apache2
```



**操作命令：**

```bash
docker-compose up -d                    基于docker-compose.yml启动管理容器
docker-compose down                     关闭并删除容器
docker-compose start|stop|restart       开启|关闭|重启已经存在的由docker-compose维护的容器
docker-compose ps                       查看由docker-compose管理的容器
docker-compose logs -f                  查看日志
```





### **docker-compose配置Dockerfile使用**

使用docker-compose.yml文件以及Dockerfile文件在生成自定义镜像的同时启动当前镜像，并且由docker-compose去管理器

```bash
version: '3.1'
services:
  ssm:
    restart: always
    build:	# build 指定构建镜像上下文、Dockerfile 文件和 ARGS 等。
      context: ../
      dockerfile: Dockerfile
    image: ssm:1.0.1
    container_name: ssm
    ports:
      - 80:80
    environment:
      TZ: Asia/Shanghai
```

创建Dcokerfile

```
from php:7.2-apache
copy wordpress /var/www/html
```

![image-20220507153540102](http://image.xpshuai.cn/2022-05-07-073540.png)

依赖docker-compose.yml启动容器

```
docker-compose up -d
```

![image-20220507153612265](http://image.xpshuai.cn/2022-05-07-073612.png)




























## LXD
特点
```
- lxd 新部署容器过程比Docker更容易
- 退出容器时不必以特定的方式退出
- lxd 容器中没有层的概念
- Docker中的层可以使部署更快(没有清理也可能感觉混乱)
```


安装
```bash
# snap适用所有发行版本
sudo snap install lxd    # sudo apt install lxd    # ubuntu官方包apt一般中不会是最新的软件包
sudo usermod -aG lxd ${USER}    # 当前用户加入lxd组，就具有管理权限了
# 重启主机
```

初始化
```bash
lxd init

# 回答问题(默认即可)

```

运行容器
```bash
# 创建容器
lxc launch ubuntu:18.04 C01 # C01是容器名称
# 自动从远程服务器下载映像并创建、运行容器(从国外下载，速度稍微慢)

# - lxd命令针对lxd管理层
# - 容器管理命令仍然使用lxc
```

常用管理命令
```bash
lxc list  # 列出所有的容器
lxc start c_name  # 启动容器
lxc stop c_name   # 停止容器
lxc delete c_name # 删除容器

lxc image list    # 查看镜像
lxc image delete  # 删除镜像

```

交互登录容器
```bash
# 执行某个容器的什么命令
lxc exec C01 bash    # root帐号登录
lxc exec C01  -- sudo --login --user ubuntu    # ubuntu镜像包含一个默认账户ubuntu
# 在本地终端直接执行容器内部的命令
lxc exec C01 -- apt update

# 退出容器(容器依然是run的状态)
ctrl+D / exit
```

随宿主机的启动, 自动启动某个容器
```bash
lxc config set C01 boot.autostart 1
```

远程映像存储服务器
```bash
# 列出远程映像存储服务器
lxc remote list    # 默认三个

- images         # 存储其他Linux发行版镜像
- ubuntu         # ubuntu自己正式发布的系统镜像
- ubuntu-daily   # ubuntu自己的非正式的系统镜像(每天更新的)
```

从远程复制映像
```bash
lxc launch images:debian/stretch test01
# 只下载映像到本地，不创建容器
lxc image copy ubuntu-daily:18.10 local: --alias u1910
lxc image list    # 查看本地的映像

# 从本地映像来创建容器
lxc launch u1910 c03
# 查看本地容器
lxc list
```


对容器进行上传下载文件
```bash
# 直接在宿主机器上执行
lxc file pull c01/etc/hosts . # 把文件下载到当前目录
lxc file push hosts c01/tmp/  # 上传文件到容器中路径

# 当然，也可以进入交互shell复制  lxc exec C01 bash
```


映像自动更新间隔(小时)
```bash
# 每隔24小时自动更新映像
lxc config set images.auto_update_interval 24
```


自动清除未使用的映像(天数)
```bash
lxc config ser images.remote_cache_expiry 5
```

查看配置
```bash
lxc config show
```


实例
```bash
# 安装apache2
lxc exec c01 bash
apt update && apt install apache2
ip addr show
curl 1.1.1.1

```

外部网络对容器内部访问
```bash
- 防火墙规则路由流量
- 创建配置我呢见DHCP 获取物理网络地址(类似VM)
- 宿主机创建桥接网卡 br0
```









## k8s

































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E5%AE%B9%E5%99%A8/  

