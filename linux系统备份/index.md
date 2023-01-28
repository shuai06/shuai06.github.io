# linux系统备份

  
## 系统备份
  
### 怎么备份

###### 常见的备份工具

##### **0.Ghost** 

---



##### 1. DIsks      (ubuntu系统查找： disks,   缺点:过程中不能压缩空间，原来多少现在备份文件就是多少)

**备份：** 选择硬盘右上角“create disk image”,  选择存储目录, 点击"start..."



**还原：** 选择硬盘右上角“restore disk image”， 选择备份好的硬盘文件，“start”

---



##### 2. Clonezilla      (备份过程中实现镜像)

- 源自台湾政府安全部门的开源项目
- 支持网络以及外置存储设备
- 服务器客户端部署

- https://clonezilla.org
- 适合大批量的...



### 下面是clonezilla的 实例：

#### 1. Clonezilla  的备份

**下载**

主要是两个版本，**两个稳定版本**和下面两个测试版本：

| Live release                                                 | Extra info                                                   | Other notes                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [**alternative stable** - 20191024-eoan](https://clonezilla.org/downloads/download.php?branch=alternative) | [checksums](https://clonezilla.org/downloads/alternative/data/CHECKSUMS.TXT), [checksums gpg](https://clonezilla.org/downloads/alternative/data/CHECKSUMS.TXT.gpg),[changelog](https://clonezilla.org/downloads/alternative/changelog.php), [known issue](https://clonezilla.org/downloads/alternative/known-issues.php), [release note](https://clonezilla.org/downloads/alternative/release-notes.php) | Ubuntu-based, [?](http://drbl.org/faq/fine-print.php?path=./2_System/57_why_ubuntu_based_clonezilla_live.faq#57_why_ubuntu_based_clonezilla_live.faq) |
| [**stable** - 2.6.4-10](https://clonezilla.org/downloads/download.php?branch=stable) | [checksums](https://clonezilla.org/downloads/stable/data/CHECKSUMS.TXT), [checksums gpg](https://clonezilla.org/downloads/stable/data/CHECKSUMS.TXT.gpg) [changelog](https://clonezilla.org/downloads/stable/changelog.php), [known issue](https://clonezilla.org/downloads/stable/known-issues.php), [release note](https://clonezilla.org/downloads/stable/release-notes.php) | Debian-based, [?](http://drbl.org/faq/fine-print.php?path=./2_System/57_why_ubuntu_based_clonezilla_live.faq#57_why_ubuntu_based_clonezilla_live.faq) |
| **alternative testing** - [20191226-eoan](https://clonezilla.org/downloads/download.php?branch=alternative_testing) [20191226-focal](https://clonezilla.org/downloads/download.php?branch=alternative_testing_2) | [checksums](https://clonezilla.org/downloads/alternative-testing/data/CHECKSUMS.TXT), [checksums gpg](https://clonezilla.org/downloads/alternative-testing/data/CHECKSUMS.TXT.gpg), [changelog](https://clonezilla.org/downloads/alternative-testing/changelog.php), [known issue](https://clonezilla.org/downloads/alternative-testing/known-issues.php) [checksums](https://clonezilla.org/downloads/alternative-testing/data_2/CHECKSUMS.TXT), [checksums gpg](https://clonezilla.org/downloads/alternative-testing/data_2/CHECKSUMS.TXT.gpg), [changelog](https://clonezilla.org/downloads/alternative-testing/changelog-2.php), [known issue](https://clonezilla.org/downloads/alternative-testing/known-issues-2.php) | Ubuntu-based, [?](http://drbl.org/faq/fine-print.php?path=./2_System/57_why_ubuntu_based_clonezilla_live.faq#57_why_ubuntu_based_clonezilla_live.faq) |
| [**testing** - 2.6.5-8](https://clonezilla.org/downloads/download.php?branch=testing) | [checksums](https://clonezilla.org/downloads/testing/data/CHECKSUMS.TXT), [checksums gpg](https://clonezilla.org/downloads/testing/data/CHECKSUMS.TXT.gpg), [changelog](https://clonezilla.org/downloads/testing/changelog.php), [known issue](https://clonezilla.org/downloads/testing/known-issues.php) | Debian-based, [?](http://drbl.org/faq/fine-print.php?path=./2_System/57_why_ubuntu_based_clonezilla_live.faq#57_why_ubuntu_based_clonezilla_live.faq) |



###### case1. 只有一个硬盘：

（对分区做备份）

1.**建议**对硬盘**分区**，一个小的**只安装系统**啥啥，另一个分区**存放资料**与分区。**备份**一个分区

2.备到**外置存储设备**



###### case2. 不止一块硬盘  (这个)

对那个系统**整个硬盘**做备份





---



**下面是对新虚拟机的实验环境：** （**自己的机器不用做这一步**）

使用disk  对新硬盘格式化：点击硬盘format disk

分区：create Partition   可以直接选择最大的分区

给个卷标：  比如bak

挂载： 点击 “播放”形状的按钮





##### 下面开始 准备备份

---



1.选择使用这个clonezilla 的iso来**引导打开操作系统**

2.然后在界面选`live`就行， 第二个选项`other mode` 可以选择分辨率

3.语言、键盘等自己选择

4.选择克隆类型：

​		设备对设备，硬盘对硬盘,...

​		这里（对本机）就选**设备到image**   (如果是还原的过程，就选image-到device)

5.选择image存放位置(有本地和云或者ssh服务器)，**这里选local device**

6.回车，开始识别本地设备，设备识别完成后，再按ctrl + c

7.选择第二个硬盘(即**目标位置**)来存放image文件， 按住tab选择 `done`,   继续回车

8.接下来，建议选择第一个选项：**Beginner mode** , 即初学者（接下来的操作会有信息提示）

9.接下来，选择**save disk**（保存硬盘） /  save  part（保存分区） ， 然后选择保存的名称

10.选择**备份源**： tab来选择第一个硬盘（即需要备份的系统盘）

11.下一步，选择备份过程是否进行检查，**推荐不检查* ** （即-sfsk） 

12. 下一步，是否再对生成的影像文件 进行检查  yes/no , **一般不用检查**， tab选择no

13.选择 加密(enc)/**不加密（senc**）, 即是否在还原的时候需要输入密码

14.接下来，选择备份之后是关机(poweroff)还是**重启(reboot)**？ 按回车进行下一步，确认无误后按住yes

15.......开始备份......  （时间大概10分钟之内）

16....备份完**自动重启**，**挂载**上硬盘，**就会看到那**个备份映像的**文件夹**



----------------

#### 2. Clonezilla  的还原

插入光盘到光驱，即clonezila的iso文件

使用Clonezilla引导，选择live，（同上）

start clonezilla

扔选择device -> image

选择Local device

他自动识别硬盘，   ctrl+c结束识别

接下来自己选择磁盘映像的硬盘的分区，选择映像文件所在的文件夹，选择done

选择 beginer mode

选择 **restoredisk** 选项：

选择ok

选择sda（即还原到哪）

是否检查选项，

欢迎之后选择是否关机/重启

回车  回车  确认无误后输入yes回车

......开始恢复......

























---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/linux%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD/  

