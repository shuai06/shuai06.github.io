# ubuntu server-存储管理






> 基本篇

# 存储
>对磁盘的所有操作都要小心小心再小心  

- 磁盘是计算机中最主要的存储设备
- 传统磁盘分区与逻辑卷管理
- 事前合理规划很重要
- 对磁盘的所有操作都要小心小心再小心


# 磁盘空间管理

- `df `  查看硬盘总体使用情况
```bash

df

"""
第一列： 文件系统
第二列： 存储空间（多少数据块）
第三列： 使用量
第四列： 可用量
第五列：使用率
第六列：挂载点
"""

df -h   # 人性化的方式

df -i    # 查看inodes使用量


```

- `du`   查看详细信息
>是谁拿走了我的存储空间

```bash
du   # 查看所有

du /   # 查看根目录/目录下的文件大小

'''
结合sudo权限
'''

'-h   # 人类易于阅读方式'
sudo du  /  -h    

'- s   # 汇总'
sudo du  /*  -hs    # 显示汇总结果

'-c   # 除了子文件和子目录之外，外加一个总和 '
sudo du  /usr/*  -hsc

```

- 第三方工具 `ncdu`   （比较人性化的工具）
```bash

apt install ncdu  # 安装


ncdu    # 使用（可以方向键和回车进入返回从 查看）
按住't'  先显示目录，再显示文件
按住'g'  显示百分比
'q'退出

```


# 添加硬盘
>扩容

用vbox模拟添加硬盘：存储->SATA->添加硬盘



###### 1.磁盘设备名称

`dmsg`命令查看设备日志

`/dev/`目录下面可以查看设备的名称

`ls /dev/sd` 

###### 2.硬盘分区

```bash
## 1. fdisk
-m  帮助菜单（查看参数）

# 查看所有硬盘设备和分区情况
sudo fdisk -l

# 分区
sudo fdisk /dev/sdb   # 进入向导
# 先创建分区表
O   # dos， MBR分区（基本分区） 传统分区格式，最大四个(主)分区，2T容量限制
g   # GPT     未来标准，128个分区，无容量限制，推荐使用
# 会生成一个16进制的guid

# 再创建分区
n
输入数字编号（后门会递增）
第一个扇区在哪个位置【让扇区决定磁盘大小】
输入结束扇区（默认最后一个扇区）
【也可以直接输入100M这种方式】

# 再保存分区
w
# 查看分区结果
sudo fdisk -l




```

###### 3.文件系统格式化
`mkfs` 命令
```bash

sudo mkfs.ext4 /dev/sdb1
sudo mkfs.xfs /dev/sdb2

```
**Linux上常用的格式：**
ext4 （普通日常使用推荐）
xfs （适用与超大空间比如100T, 多文件，数据库）

###### 4. 挂载
>绑定访问路径

**一般两个挂载目录:**
`/mnt/`  一般是硬盘等  （下面创建空的子目录再挂到子目录下）
`/media/`   一般习惯使用挂： u盘等移动存储

1.`mount`命令   (临时性的，重启之后失效)
```bash
# sdb代表那块硬盘， sdb1代表那个分区
sudo mount /dev/sdb1 /mnt/storge/

# -t 声明格式
sudo mount /dev/sdb1 -t ext4 /mnt/storge/

# 卸载目录下面的设备（如果busy就先退出这个目录先不要使用）
sudo umount /storage

```

2. 自动挂载 `/etc/fstab`
```bash

'''
第一列：设备ID         UUID， 唯一的标号/ 设备名也可以，推荐id
第二列：挂载点         swap无需挂载(none)
第三列：文件系统      ext4, swap...
第四列：options     （ 参数， error=remount-ro的意思是发生错误只读形式挂） 一般填写默认就行 defaults:rw,exec,auto,nouser.asynchronous（异步存储）           nouser表示当前用户才可以
第五列：dump        （基本没用，写0不备份就行，   备份的时候才设置1）     一般写 0
第六列：pass            (FS错误时fsck是否检查错误，0不检查，1优先，2后查）  一般写 2

'''
实践：
1.
blkid  查看设备UUID
lsblk -fP  查看设备id

2.编辑文件

3.
reboot   
或者
mount -a   #将/etc/fstab的所有内容重新加载


```

-------------------------------------------------
>高级篇

# SWAP管理
> 可能是你的救命稻草

- 关于swap的争论
    - 需要/不需要  （Raspberry Pi）
    - 大小多少才合理（2-16G）
    - Swap分区，swap文件
- 理想状态下应该无事可做的swap
    - 大量的swap空间使用意味着服务器已疲于奔命
    - free -m      free -h     查看内存以及swap的
- 某些云主机默认没有swap  


ubuntu中，是swap文件  

###### Swap文件
/swapfile


#创建文件 制定大小，位置
`sudo fallocate -l 8G  /swapfile`
#查看文件
`sudo xxd  /swapfile`
#格式成swap文件  (记下 UUID)
`sudo mkswap  /swapfile`
#把权限调小
`sudo chmod 0600  /swapfile`
#修改挂载文件
sudo vim /etv/fstab/
        `/swapfile none swap sw 0 0`
#关闭原来的swap
`sudo swapoff -a`
#开启
`sudo swapon -a`



###### Swap分区

1.新增加一块硬盘
（如果vox提示已经创建过了，打开目录，删除那个硬盘[不要误删主硬盘和快照]，和.pre文件； 编辑.vobx文件，把新增硬盘配置项删掉）  

`ls /dev/sd*`

2. 分区
`fdisk /dev/sdb`

3.新建主分区，`t`修改分区表类型为swap分区的格式 `82`，`w`保存退出  (L 列出所有支持的分区类型)
可以看到：`/dev/sdb`已经分出了`/dev/sdb1`

4. 格式化为swap
sudo mkswap /dev/sdb1

5修改挂载文件
sudo vim /etv/fstab/
        `UUID=xxxxxx none swap sw 0 0`
        
6.关闭原来的swap
`sudo swapoff -a`

#7.重新开启
`sudo swapon -a`

原有的swap就可以删除啦～～～



# LVM管理
>一项值得感谢的技术


- 无须重启计算机灵活调整硬盘空间大小
    - 硬盘分区太大浪费，太小无法满足业务发展需要
    - 将传统的硬盘分区逻辑的组合为资源池，按需分配
- 概念
    - Volumn Groups  （卷组）（池化）： 把物理资源的变成逻辑的
    - Physical Volumns (物理卷) 
    - Logical Volumnes （基本分区、RAID）： 可以存储数据的逻辑的概念，VG一部分资源的分割  
- 安装包
    - sudo apt install lvm2

添加硬盘， 先创建PV, 再创建VG, 再创建LV

**1.物理卷：**
```bash
# 创建物理卷
sudo pvcreate /dev/sdb
# 查看物理卷
sudo pvdisplay
# 查看简略的摘要信息
sudo pvs


```

**2.卷组：**
```bash

sudo vgvreate vg01 /dev/sdb
# 可以一次性指定多个pv
sudo vgextend vg01 /dev/sdc /dev/sdd

# 查看概要信息
vgscan / vgs

sudo vgs

```
**3.逻辑卷**
```bash
# 创建逻辑卷， -n 名称, -L 大小， 从哪个vg里面分割的
sudo lvcreate -n lv-database -L 1G vg01
    # 查看简要信息
    sudo lvdisplay
    sudo lvscan / lvs

# 格式化，(fstab自动)挂载
sudo mkfs.ext4 /dev/vg01/lv-database
    sudo mount /dev/vg01/lv-database/  /mnt/lv-01
    df -h

# 灵活拓展的特性
sudo lvextend /dev/vg01/lv-01 -l  256   # 增加完毕后大大小 256
sudo lvextend /dev/vg01/lv-01 -l +256  # 带加号，又增加大小 256个块

sudo lvextend /dev/vg01/lv-01 -L +2G    # 增加2G

# VG FREE  谁的百分比
sudo lvextend /dev/vg01/lv-01 -l 50%VG   # 拓展到vg的50%
sudo lvextend /dev/vg01/lv-01 -l +50%VG   # 再拓展vg的50%

# 逻辑卷扩充了
df -h  # 是文件系统的查看。 这之后还没有告诉文件系统
# 把逻辑卷的扩充告诉文件系统，就可以用啦
sudo resize2fs /dev/vg01/lv-01

```

**逻辑卷缩小**
```bash
umount /dev/vg01/lv-01  # 必须下线
e2fsck -f /dev/vg01/lv-01  # 检查文件系统
resize2fs /dev/vg01/lv-01 1G # 缩小文件系统
lvresize /dev/vg01/lv-01 -L 1G  # 缩小逻辑卷

# 重新挂载查看
sudo mount /dev/vg01/lv-01 /mnt/lv-01/
```

**快照**
- 故障时可回退
- **临时可回退机制，不可视为备份** (备份应该是异地备份)
    - 本地保存，安全性无法保证
- 系统根目录使用快照可以用于测试补丁更新
    - 测试完成合并并删除快照
- 创建快照等同创建一个新的LV（默认是空的）
    - 初始不占空间，但文件发生修改时原块数据被拷贝
    - 回退时将快照中原始数据覆盖当前快照已经被修改的

```bash
# 快照  -s 是快照lv，   -L大小
sudo lvcreate -s -n s01 -L 2G vg01/lv-01
# 快照LV可以被直接挂在，用于恢复单个文件

# 恢复快照（恢复后快照被删除）  --mergr 哪个快照
sudo lvconvet --merge vg01/s01

# 直接是不能恢复的，我们查看一下
sudo lvscan  # 让他不是active才行

sudo umount /mnt/lv-01  # 退出当前目录，执行此命令卸载目录
sudo lvchange -an vg01/lv-01  # -n 让他不active
sudo lvchange -ay vg01/lv-01  # -y 让他active
sudo mount dev/vg01/lv-01  /mnt/lv-01   # 重新挂载

```
**移动物理卷数据**
当磁盘性能或老旧等因素需要更换硬盘，提前转移数据
`# 后门是目标盘，前面 -n 是指定lv数据,   -i 数值，每隔几秒显示一下运行状态  `
`sudo promove -n lv-01 /dev/sdb  /dev/sdc -i 1`
`sudo pvdislpay`



------

> 从业务的角度来思考技术，不要做书呆子技术人员

------
# RAID 高级管理
> 与LVM结合将达到完美


**`软RAID`基于`mdadm`驱动实现**
软RAID并不比硬RAID差
```bash
sudo apt install mdadm
sudo mdadm -C -v /dev/md0 -l1 -n2 /dev/sdb /dev/sdc -z100M -x1 /dev/sdd
-C    创建
-v    详细信息
-l    RAID类型（0，1，5，6，10）
-n    RAID盘数量
-z    从每个硬盘占用多少空间创建RAID
-x    Spare盘（备用盘）数量+盘名

# 约定俗成： 逻辑设备：以0结尾，物理设备以1结尾
```

**常用管理命令：**

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200207110021066_291689330.png"> </img>

查看raid盘的状况：    `cat /proc/mdstat` （开机之后临时生成的）
-r 删除物理硬盘（如果正常状态下是提示busy的， 可以-f设置失效, --re-add重新加入磁盘阵列）
-A 手动出发数据同步

- 替换硬盘后安装Grub(系统启动盘)
`sudo grub-install /dev/md0`

- 格式化和加载
`sudo mkfs.ext4 /dev/md0`

------
# 链接
- 符号链接(软链接) 和 硬链接
    ls -li （查看inode值）
    inode: 存放文件元数据的数据对象，表现为一个数值编号


- 创建`硬链接`（inode值是相同的， 给不同数据起了不同的名）
    ln fileS fileD
    ln log2013.log  ln2013

    目录不能创建硬链接
    不能移动硬链接到不同分区（inode改变）

- 创建软`链接`
    不共用inode
    ln -s fileS fileD （可以跨设备，可以给目录... , 相对路径和绝对路径， 建议源文件是绝对路径，目标链接是相对路径）

---

# 数据加密/磁盘加密
> 数据最值钱  

**清除所有数据：**
```bash
# 把磁盘所有字节数据全覆盖为0
sudo dd if=/dev/zero of=/dev/sdc
```
**磁盘加密：**
```bash
# 安装软件包
sudo apt install cryptsetup             
# 加密分区（需提前分区 fdisk）  （boot/引导分区是不能加密的）
sudo cryptsetup luksFormat /dev/sdc1
# 打开加密分区（随意命名）
sudo cryptsetup luksOpen /dev/sdc1 srypt1
# 格式化加密分区
sudo mkfs.ext4 /dev/mapper/crypt1
# 挂载在加密分区
sudo mount  /dev/mapper/crypt1  /mnt/crypt1
# 更改所有者
sudo chown xps:xps /mnt/crypt1
# 关闭加密分区(先umount)
sudo umount /mnt/crypt1
sudo cryptsetup:luksClose crypt1

```

> 不仅对硬盘有用， 也可以对U盘进行加密
可以用cryptsetup对u盘加密，设置一个足够强度的密码， 也可以再格式化为ext4格式，让win用户不能直接打开




















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E5%AD%98%E5%82%A8%E7%AE%A1%E7%90%86/  

