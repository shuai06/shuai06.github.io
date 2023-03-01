# ubuntu server-引导过程




## BIOS/UEFI
>打开黑箱看看里面

操作系统是如何启动的
###### 三个彼此独立但相互关联的过程
1.Bios/UEFI自检过程（POST）
```bash
- 加电自检
- 硬件识别与检测

# UEFI比较新，原理和效果不一样
```
2.Boot Load加载执行过程
```bash
引导加载程序允许您选择想要引导的操作系统
```
3.操作系统启动过程
```bash
启动各种服务及操作系统本身
```

###### BIOS
>打开黑箱看看里面

主板芯片（计算机运行的第一个程序）
```bash
- 执行基本的检查或开机自测(POST)
- 识别连接在计算机上的所有硬件、可用性、底层参数
        例如内存、硬盘、键盘和显卡等
- 不同的BIOS可以设置检查参数
- 轮训硬盘控制器，启动其板载芯片，配置RAID等
- 引导顺序
- 不当的配置可导致计算机无法使用

```

###### UEFI
>打开黑箱看看里面

较新的主板支持UEFI (2015年以后)
```bash
- 视觉上与BIOS相似，但原理极不相同
- 运行速度略快于BIOS
- 创建UEFI是为了克服BIOS的故有缺陷
    - BIOS运行基于16位处理器，UEFI基于现代处理器
    - 内存使用量方面，BIOS有 限，UEFI按需要量使用
    - BIOS只能读取MBR引导操作系统; UEFI可读取系统安装时划分的独立分区
    - BIOS支持最大2T的硬盘，UEFI使用GPT分区表，最大支持8-9Z硬盘

```

###### 安全引导(Secure-boot)
>打开黑箱看看里面

```bash
- 恶意软件可通过Boot Loader加载到系统中
- UEFI提供了安全引导 Secure-boot
- 加载前通过公钥/私钥验证签名的引|导加载程序和硬件
- 64位操作系统支持Secure-boot,某些硬件无法更改操作系统类型（如某些电脑厂商不让电脑装其他系统，他给锁死了，要问好Secure-boot能不能关或在开启Secure-boot时候能不能按其他系统。其实建议Secure-boot开启）
- 目前绝大多数硬件同时支持BISO/UEFI，可自行选择启动

```

###### 硬盘分区表
>打开黑箱看看里面

```bash
- 分区表通常位于硬盘起始位置
- 记录硬盘的划分情况
    - 两种类型：MBR / GPT
```
<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%86%85%E6%A0%B81.png" />


## Boot Loader
>打开黑箱看看里面

- BIOS/UEFI不关系统安装的操作系统
- POST自检结束后，BIOS/UEFI读取硬盘Boot Loader
- 读取并执行Boot L oder，由其引|导操作系统
- Linux世界里两大`Boot Loader程序`
    - LILO:基本已被废弃
    - GRUB:目前绝大多数发行版采用


###### BIOS引导过程
>打开黑箱看看里面

BIOS -> Boot Loader (`GRUB2`)
第一阶段：boot.img读取并运行`MBR`硬盘起始的446字节  
第二阶段：boot.img 查找并运行core.img (通常位于MBR Gap)  
                 core.img的任务是访问`/boot/grub`，加载该目录下的所有模块  
                 加载启动菜单，自动或手动选择进一步引导的系统内核 


###### UEFI引导过程
>打开黑箱看看里面

第一阶段
- UEFI读取`GPT`分区表，找到`EFI分区`(格式Fat32 / 大小默认537M / boot esp )
- 从中读取、执行boot loader代码，加载模块
- 命令`parted` 一`print` (读的是文件：/boot/efi/EFl/ubuntu/ grubx64.efi)
```bash
sudo parted    
# 查看 分区的信息
print

# disk 工具也可来查看分区

```

第二阶段
- 查找`/boot/grub/x86_ 64- -efi/core.efi` 执行
- 生成GRUB2菜单， 选择不同内核引导系统(内核二进制文件:`vmlinuz-<release- number>` )
- 加载硬件驱动文件initrd.img到内存中，然后控制权交给内核，继续引导
- 系统加载systemd / upstart / init, 启动各服务



#### GRUB2
GRUB是一个强大的多引导加载程序
```bash
- Windows --下的引导程序--> ntldr    # 不兼容Linux
- MacOS ---> Boot Camp
```

两种启动方式
```bash
-直接查找和加载所需的内核
-加载另一个引导加载程序
```

操作系统引导的四个核心组件
```bash
内核文件、驱动器名称、内核文件所在分区号、初始化RAM盘
```


###### GRUB菜单
**GRUB菜单文件**
```bash
/boot/grub/grub.cfg (自动生成，不可修改)
主要来源文件/etc/grub.d/*

菜单定义: menuentry / submenu

```

**显示菜单**
```bash
- Vi /etc/default/grub 
    GRUB_TIMEOUT_STYLE=menu    # 让他显示菜单
    GRUB_TIMEOUT=100    # 超时时间（100s如果不做选择，就启动默认的）
 sudo update-grub
```

**单用户模式**
类似于Windows启动时F8灾难恢复模式
BIOS: Shift
UEFI: Esc

GRUB菜单
-“`e"`
-在引导项最后增加`“single"`
-` Ctrl+ X`    进入



手动生成GRUB菜单文件
```bash
sudo grub-mkconfig --output mygrub.cfg
```

**安全风险**
```bash
恶意用户可能修改boot loader, 造成安全问题
```

设置GRUB密码
```bash
sudo vi /etc/grub.d/40_custom    # custom的文件在更新系统后不会被覆盖
    set superusers=xps
    password yuanfh pass123
sudo update--grub

```
以上明文存储GRUB密码存在风险隐患
```bash
# 使用工具 grub-mkpasswd-pbkdf2
grub-mkpasswd-pbkdf2    # 输入密码，会生成一段密文
sudo vim /etc/grub.d/40_custom
    password_pbkdf2 xps grub.pbkdf2.sha512 ......    # 上面的密文放到配置文件中
    
sudo update--grub

# 此方法不适合远程托管到IDC机房的机器（一重启在grup时就要输入密码，此时SSH还未连接）

# 指定启动项用户
menuentry --> --users xps
```



#### 其他BootLoader(LILO, Grub之外的)
**1.SYSL INUX**
```bash
从FAT分区引导系统的BootLoader (U盘启动时)
```

**2.EXTL INUX**
```bash
# 小型的Linux
从ext2、ext3、 ext4、 btrfs引导系统的小型BootLoader
```

**3.ISOL .INUX**
```bash
# 光盘启动
从LiveCD、LiveDVD引导系统的BootLoader
# isolinux.bin 映像文件; isolinux.cfg 配置文件
```

**4.PXELINUX**
- (无盘站-->无硬盘)`从网络服务器引导系统`的BootLoader
- PXE（Pre-boot eXecution Environment）
- 使用DHCP为工作站分配IP
- 使用BOOTP加载BootLoader映像
- TFTP将引导映像传输至工作站(基于UDP协议)
- `/tftpboot/ pxelinux.0` : PXE BootLoader
- `/tftpboot/ pxelinux.cfg` :配置文件
- 目前新版支持NFS、HTTP、FTP服务器


#### 系统无法启动!
**两大问题原因**
- 内核问题：手动安装内核时缺少模块、库文件(发行版错误)
- 驱动器故障：无法读取根驱动器

**处理办法:**
- 使用先前无故障版本内核引导系统
- 单用户模式(指定内核参数)
- 对于驱动器故障，可以使用救援盘(CD/DVD、USB) 检修硬盘：`fsck /dev/sda1`

**显示GRUB菜单**
- 可以开机`一直按住ESC键、或SHIFT键`（在故障发生时，且没修改过grup菜单显示时）



## 操作系统启动
>服务/Service (windows) ==后台/daemon (Linux)

INIT：内核引导后运行`第一个程序INIT`,初始化操作系统并启动服务(直到关机)
    - 内核加载后的初始化进程，用于启动其他程序
        - 所有服务启动由init 程序处理(`PID=1`) ，所有系统服务进程都是其子进程
        - `pstree -p 1`  
        - `/usr/sbin/init -> /lib/ systemd/systemd`  现在的`init`程序早已不是以前那个init，只是沿用，实则是个符号链接
        - 早期版本配置文件：`/etc/inittab` (`ubuntu无此文件！`)

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%BC%95%E5%AF%BC%E8%BF%87%E7%A8%8B2.png"/>

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%BC%95%E5%AF%BC%E8%BF%87%E7%A8%8B3.png"/>


## 三种INIT
>接上面

**1.SysV / SysV-init**
源自UNIX System V (目前老版本Linux还在使用)

**2.Upstart**
兼容SysV，Ubuntu意图用于替换SysV，其他发行版也有使用
几乎没有完整实现的Upstart（Ubuntu自己都不用Upstart了）

**3.Systemd**
目前主流发行版使用`systemd-sysv`包兼容SysV格式的initd脚本
始于2010年，是目前最新的系统初始化后台程序;
速度更快，效率更高


#### SysV
- 分成不同运行级别的一组Shell脚本(哪些程序，什么时候运行)
- 每个程序有一个独立的脚本控制其启动或停止
- 系统启动时进入一个运行级(一组随系统启动的程序)
- 内核启动首先读取运行级别配置文件，决定进入哪个运行级
- 运行级别：
```bash
0  关机
1  单用户模式(系统维护)
2  Debian多用户图形模式、
3  多用户文本模式
4  未定义
5  RedHat多用户图形模式
6  重启系统
# 不同发行版可能略有不同
```

###### 在不同运行级下启动程序的方法
**方法1：`/etc/ inittab`：定义不同运行级下启动的程序，每行定义一个应用程序**
```bash
id:runlevels:action:process
#- ID: 包含1-4个字符，唯一标识一个程序
#- runlevel: 运行级列表(在哪个运行级中运行该程序，例如: 345)
#- action:
#- id:3:initdefault: 系统启动后默认进入的运行级
```

**方法2：启动脚本**
`/etc/init.d/`下的脚本
`/etc/rc1.d`为例是不同级别1下要运行的程序(符号链接)，`K`开头表示kill掉,`S`是启动，后面数字的值代表优先级，最后面代表程序的名称

除了运行级，SysV也使用启动脚本控制程序启动、停止
启动脚本存放于`/etc/init.d/`中，通过`/etc/rcX.d/`目录调用(X 运行级)
脚本文件名`S: Start`; `K: Kill/Stop`，` 数字表示优先级`(依赖顺序)
通过具体脚本启动停止程序稍嫌麻烦，系统包含工具指定脚本的运行级
```bash
# 给我们提供了管理工具
chkconfig / update-rc.d  #用于查看和修改程序的运行级(RedHat / Debian)
chkconfig --list <program_name>    #查看
chkconfig -- levels 12345 <program_name> on    #设置
update-rc.d program defaults / update rc.d <program_name> remove
update- -rc.d -f <program_name> start 40 2 34 5. stop 80 0 1 6. #具体运行级别和顺序(在那几个级别启动这个程序，在那几个级别stop这个程序)  40、80指在具体运行级中程序的启动顺序(0-99)
```


**action字段**(在Ubuntu中已经找不到了)
- boot  进程在系统扃动时启动
- bootwait  进程在系统启动时启动，系统会等待它启动完成
- initdefault  系统默认进入的运行级别
- kbrequest  进程在按下特殊的组合键后启动
- once  当进入运行级别时，进程启动一次，后面down了也就不管了
- powerfail  系统关闭时才运行
- powerwait  系统关闭时才运行，系统将等待其运行完成
- respawn  进入运行级时启动，并在终止时重新启动（病毒可以这样，你杀掉之后过一会又会起来）
- sysinit  在boot和bootwait项之前启动
- wait  进程启动一次，系统等待其完成
  

**查看系统运行级**
```bash
runlevel    # 显示之前和现在的运行级
N  5    # 上面命令的执行结果
# N    表示系统【之前】处于原始boot运行级
# 5    表示【当前】运行级
```
**更改运行级**
```bash
init 6    # 重启系统(teljinit)
init 0    # 关机

# init的问题在于命令立刻生效。这对个人电脑没有问题，但对多用户环境有影响（可能造成其他用户也...）
# 多用户环境，建议使用shutdown、halt、 poweroff、 reboot (时间、通知)

## 接受别人给发的通知
mesg y    # 开启
mesg    # 查看

who -T    # 第二列终端前面如果有+号，表示他允许别人给他发消息

write xps tty1     # 给指定用户在某个指定的终端发送消息
#运行后直接写消息就行

wall xxxxxxx   # 给所有登录用户发消息（广播）

```


#### Upstart
- 操作系统的复杂度不断上升，造成SysV脚本越来越复杂
- ubuntu官方开发Upstart用于替换SysV
    - 处理可热插拔设备在Linux系统中造成的动态环境
    - 使用公钥`/etc/init/`文件夹替代/etc/inittab 和/etc/init.d 中的启动脚本
    - 每个程序和服务在init文件夹中都有一个`<program_name>.conf`配置文件
    

启动和停止服务
```bash
sudo start/stop bluetooth        # ubuntu中没有
```



**Linux Standard Base (LSB)**
- 是一些Linux发行版协商共同支持的规范
- 目的是在不同Linux发行版之间创建一 致的使用体验
- LSB定义了标准的init命令:·`start、 stop、 restart`
- 所有遵循L SB的Linux发行版都应该支持以上命令
- 但是Debian等发行方退出了LSB组织，目前ubuntu中不包含
```bash
lsb_release -a    # 
```

Linux世界可能会日趋分裂，统一用户体验的缺失势必提高用户的学习成本


#### Sytemd(目前最主流)
- 由RedHat开发，并迅速流行于Linux世界
    - Systemd 在系统初始化的方式上引入了一个主要的范式转换，并引起了一些争议
    - 它放弃使用多 个小型初始化脚本，转而`使用单一程序`，配合每个服务独立的配置文件
        - 这违背了早期的Linux哲学(小而专著是性能、稳定、安全性的基础)

- 放弃了初始化脚本和运行级，Systemd创造了`target / unit`的概念
    - `unit`用来定义service、action
    - 由名称、类型、配置文件组成
    - 常见unit类型
    ```bash
    automount、 device、 mount、 path、 service、 snapshot、 socket、 target 等等
    ```

unit的命名方式
```bash
name.type    # 如sshd.service
```

当前系统已加载的unit
```bash
systemctl list- -units     # 查看
```

Systemd 使用service类型的unit 管理后台服务

target 类型组合多个unit实现同时启动
```bash
例如network.target组合了所有用于启动网卡的unit
系统初始化过程中，target 类似于SysV中运行级别的概念
一个target对应- -组service
```

**配置unit**
每个unit需要一个配置文件来定义需要启动、如何启动什么程序
配置文件路径`/lib/systemd/system/`，不同版本，路径不尽相同
```bash
cat /lib/systemd/system/ssh.service
    ExecStart    #指定运行程序
    After    #依赖的服务(先于ssh服务运行的程序)
    WantedBy    #指定target (运行级)
    Restart    #何时重启
```


tartget配置文件
```bash
定义需要启动的unit(不定义具体程序)
# 以graphical为例
cat /lib/systemd/system/graphical.target
```

默认target
```bash
# 系统引导后默认启动的target(default.target)
/lib/systemd/system/default.target -> graphical.target    # 只需对目标做个软链接
```

systemctl程序
```bash
# 常用参数
list-unites / start / stop / restart / reload /status /
enable / disable    #计算机每次引导时启动/禁用
isolate             #启动指定单元(停止其他所有单元)
default    #修改默认target
```

兼容SysV
```bash
# 对应的运行级别
/lib/ systemd/system/runlevel0.target    # poweroff.target
/lib/ systemd/ system/ runlevel1.target    # rescue.target
/lib/ systemd/ system/ runlevel5.target    # graphical.target
/lib/ systemd/ system/ runlevel6.target    # reboot.target

# 比如，进入到运行级别1
systemctl isolate runlevel1
```



## 内核
#### Linux系统组成
- Linux内核
- GNU系统工具
- 图形化桌面环境
- 应用软件

#### 内核控制计算机软硬件
- 由L inus Torvalds(当年是赫尔辛基大学的学生)创造，
- 直到目前为止的负责人(Linux Foundation)



#### 操作系统的核心功能
- 管理系统的内存
- 管理软件程序
- 管理硬件
- 管理文件系统

- 如何安装内核(其不同的组成部分)？
- 如何创建新内核(支持新硬件和软件特性)？
- 如何管理内核及内核模块？


#### 内存管理
- 在系统的内存容量限制下，控制程序如何运行
- 除管理物理内存，操作系统还负责创建和管理`虚拟内存`
    - 虚拟内存并非真实存在，它`利用硬盘空间`创建并当作真实内存使用(`SWAP`)
    - 内核将一段时间不活跃的内容页转储到SWAP，并在需要时重新读入物理内存
    - `Swapping out机制`使系统认为自己拥有更多可用的内存空间(降低性能)
    - 内存空间被分组到称为内存页的块中
    - 内核定位物理内存或交换空间中的每个内存页
    - 内核维护内存页表，登记哪些页位于物理内存/SWAP

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.png"/>


**查看虚拟内存**
```bash
cat / proc/ meminfo    # 运行实时的数据
# 
MemTotal:    2048000 kB    # 物理内存
MemFree:    438123 kB      # 空闲
MemAvilable    3580848 KB    # 实际可用的空间
SwapTotal:     2048000 kB    #
SwapFree:    2048000 kB
```
Linux系统每个进程的内存彼此独立
```bash
不同进程无法互访彼此内存空间，内核负责维护这种约束
没有其他进程可以访问内核使用的内存空间
```
共享内存页
```bash
内核可创建共享的内存区域，供多进程共享数据，内核负责验证和准入进程
```

查看当前系统共享内存页
```bash
ipcs -m    

# 结果字段：
owner    #创建此共享内存段的帐号
perms    #共享权限
key    #允许其他用户访问此内存段
```


#### 软件程序管理
运行中的程序称为进程
```bash
内核控制所有进程运行(前台运行、后台运行)
初始化进程(init) 负责启动所有其他进程
内核每启动一个进程，都在内存中为其分配独享的一段内存区域存放数据、代码
```

查看当前进程
```bash
ps
ps -ax    # 建议加参数

## 显示的字段含义
# PID：进程ID
# STAT：进程状态(Sleep、 Sleep and Wait、Run)
# [process] swapping out    带放括号的都是放到虚拟内存中的
```


#### 硬件管理
- 与系统通信的所有硬件都需要将驱动植入内核
```bash
内核编译同时包含硬件驱动(早期唯一方法)
驱动程序以内核模块型式加载
```

- 内核模块是一个自包含的驱动程序库，可动态与内核链接/取消链接
- Linux系统的三种硬件设备文件
```bash
1. 字符设备文件: 一次只能处理一个字符数据的设备(调制解调器、终端)   c
2. 块设备文件: 一次能处理一大块数据的设备(硬盘)      b
3. 网络设备文件:使用包来发送和接收数据的设备(网卡)     
```


**Linux为系统上的每个硬件设备创建称为节点的特殊文件**
- 内核通过`唯一的数字对`标识每个节点(`主`、`次`)
- 同类型设备主节点号相同，同一主节点号下，次节点号唯一
```bash
cd /dev && ls -al sd* ttyS*
# 结果显示
brw-rw---- 1 root disk 8, 0 12月5 15:14 sda    # 8，0 主设备标识/次设备标识
brw-rw---- 1 root disk 8, 1 12月5 15:14 sda1
brw-rw---- 1 root disk 8, 16 12月5 15:14 sdb
brw-rw---- 1 root disk 8, 17 12月5 15:14 sdb1
crw- -rW---- 1 root uucp 4, 64 12月5 15:14 ttyS0
crw-rw---- 1 root uucp 4, 65 12月5 15:14 ttyS1

```


#### 文件系统管理
文件系统定义OS如何在存储设备上存储数据
- 分区、格式化
- Linux 支持包括Windows在内的众多文件系统
- 内核统一使用Virtual File System (`VFS`) 与不同文件系统进行交互


#### 内核的文件构成
**内核二进制文件**
- 内核程序本身，BootLoader加载到内存中执行的内容(位于`/boot`或`/`中)
- 基于编译打包的驱动数量不同，内核=进制文件可能很大
- 内核二进制文件通常被压缩，以便加载内核时节省内存空间
- 压缩方式不同，将导致内核二进制文件名称差异


| 不同的Linux内核文件 |                         备注                          |
| ------------------ | ----------------------------------------------------- |
| bzimage            | （常用）GNU zip压缩的大内核，常被拷贝为4(vmlinuz)        |
| kernel             | 未压缩                                                |
| vmlinux            | 未压缩，通常不用作最终的引导版本                         |
| vmlinuz            | 通用的压缩文件（如ubuntu下：vmlinuz-4.15.0-65-generic） |
| zimage             | 使用GNU zip压缩的小内核                                |
|                    |                                                       |


#### 内核模块
**内核直接与硬件交互**
- 内核与硬件通信需要驱动程序的支持
- 将硬件驱动和内核的`源码`一并编译成二进制执行文件(体积大)
- 将驱动编译为二进制的模块对象文件`.ko`， 运行时动态链接

**硬件驱动的发布**
- 源码：开源精神
- 二进制文件：保护隐私、功能特性

**内核模块(`modprob`)**
```bash
/lib/modules/<version>/kernel/*.ko
```


#### 内核源码
**获得**
- Linux内核开发库(www.kernel.org --> 稳定版、开发版)
- 对应Linux发行版的软件库(对本发行版最稳妥，下载方便)
- 下载源码解压至`/usr/src/linux` (内核工具默认查找路径)
```bash
链接到指定版本 /usr/src/linux-5.3.12    # 维护多个版本
/usr/src/kernels    # Redhat
```


#### 内核补丁
- 针对bug修复和安全补丁的增量内核发行版
- 内核补丁版本(`针对主版本的补丁`)
    - 只包含应用于主内核源代码发行版以获得增量发行版的`更改`(5.3 --> 5.3.1)
    - 使用`patch`命令将补丁源码包应用于主源码包，然后重新编译
- 后续补丁版本(5.3.1 -> 5.3.2)
    - 卸载早期版本补丁(`patch -R`)
    - patch新版本补丁
    - 重新编译



#### 内核头
Linux内核绝大部分使用C语言编写
    - C语言头文件:编译时需要的库文件
    - 编译内核、模块都需要相同的内核头(库文件)

Ubuntu内核头
```bash
sudo apt install linux-headers-4.15.0-72-generic
/usr/src/linux-headers-4.1 5.0-72-generic/include/config/*.h    # ubuntu
/usr/src/kernels    # Redhat
# 不同发行版路径不同

# 查看使用的内核版本
uname -r
```


#### 内核文档
- 许多独立的文本文件，说明每个源代码文件在内核结构中的作用
- 针对内核源码做任何操作之前，建议详细阅读文档
- 由于文档体量巨大，通常以独立包形式发布
- 文档路径
```bash
/usr/src/linux/Documentation    # Ubuntu
/usr/src/kernels    # Redhat
```


#### 内核版本
- 七个内核版本命名方式(系统)
- Linus于1991年9月发布了初始Linux内核版本号<font color=red>0</font>.01
    - 0表示该版本目的是测试，并非生产环境使用(延用至0.95 -- 1 992.3)
- 1994年3月，Linus 发布了Linux 的第一个生产版本<font color=red>1</font>.0 (`1.x.y`)
    - x主版本号(`奇数`表示测试/开发版本；`偶数`表示稳定/生产版本)
    - y主版本下的补丁版本
    - 此版本号延用至 1.3 (1995.5)

- 1.3之后Linux内核变化很大，因此下一个版本为<font color=red>2</font>.0 (1996.6)
- 2.x.y延用1.x.y 的命名方式，直至2.4 (2001.1)
- 2003.12 Linus发布2.6.0版本，启动了一个新的版本命名系统
    - 由于2.6内核非常稳定，因此新内核版本格式为2.6.x.y（`不再通过奇偶数来区分稳定与否`）
    - 2.6 每个版本都是生产版本(`开发版结尾为-rc`)
    - 此版本延用至2.6.39 (2011.5)

- 2011.7 Linus为庆祝Linux内核20岁生日，启动了内核的3.0版本
    - 3.x.y (开发版在结尾加-rc) -直延用至3.19 (2015.2)
    - 3.x.y-<font color=red>z</font> 临时特殊补丁版(主要与紧急的安全修复有关)
- 2015.4发布4.0版本
    - 4.x.y (与3.0相同)
    - 4.x.y-<font color=red>z</font> (y的第z次`微调`版本)
- 2019.3发布5.0版本
    - 目前最新，命名延用以前

查看内核版本
```bash
uname -r
```


#### 维护内核
- 手动编译内核(较少使用，忽略)
- 使用发行版包管理器维护内核（建议的方式）
- 模块文件
```bash
/lib/modules/version/kermel/*.ko
```
- 手动编译内核模块，当升级内核时需要重新编译模块
    - Dynamic Kernel Module Support (`DKMS`)
    - 注册自定义模块后，dkms监视内核变化，并自动运行脚本重新编译安装模块



#### 常用模块命令
```bash
# 已安装模块列表
lsmod    # 模块之间存在依赖关系

#  查看模块详细信息
modinfo bluetooth    # 比如查看蓝牙模块的信息

# 安装/删除模块
insmod / rmmod    # 需指定'完整'内核模块文件路径(依赖模块未加载时失败)
modprobe -r    # 或者这种方式安装，只需指定模块名，自动安装依赖（比较友好）
```


#### 内核排错
**查看内核版本**
```bash
uname -r    # -a显示全部信息
```

**/proc目录**
```bash
# Linux内核创建的动态伪目录
# 在内核运行时查看内核相关的配置、性能
# 包含硬件信息

/proc/interrupts    # 被分配的中断信息
/proc/ioports    # I0端口使用情况
/proc/dma    # DMA

lsdev    # 查看以上综合参数的命令，apt install lsdev
cat /proc/sys/kernel/version     # 查看内核参数


## 修改内核参数
cat /proc/sys/net/ipv4/ip_forward/        # 如：开启路由功能，查看
echo 1 > /proc/sys/net/ipv4/ip_forward    # 如：开启路由功能，修改

sysctl net.ipv4.ip_ forward1    # 或者使用命令，查看
/sysctl -W net.ipv4.ip_ _forward=1   # 使用命令开启，修改

/etc/sysctl.conf    # net.ipv4.ip_ forward = 1    修改配置文件，让它永久生效

# 其他发行版可能在这个目录修改
/etc/sysctl.d    

```

























---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E5%BC%95%E5%AF%BC%E8%BF%87%E7%A8%8B/  

