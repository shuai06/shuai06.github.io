# ubuntu推荐生产力软件


  
  
### 基础软件包

```
sudo apt intall flashplugin-intaller     # 安装浏览器flash插件

sudo apt install meld    # 对两个目录/文件 一一比对

sudo apt install amule   # 电驴

sudo apt install transmission  # BT下载工具 （还可以）
sudo apt install qbittorrent   # BT下载工具

sudo apt install ttf-wqy-microhei  # 字体文件

sudo apt install mtr  # 网络路径追踪工具（基于图形化）

sudo apt install whois

sudo apt install git

sudo apt install curl  # 命令行完成对浏览器访问的基本操作（写脚本常用）

sudo apt install obs-studio # 录屏 （图标是旋风形状的，也可以直播）

sudo apt install ubuntu-restricted-extras # 一组视频的解码器
sudo apt install libavcode-extra libdvd-pkg # 使得linux可以比方dvd的介质。配合上面那个

sudo apt install unrar unrar-free  # 解压rar文件

sudo apt install woeusb  # 将iso文件刻录到u盘上面

sudo apt install ascii # 运行命令查看常用的ascii编码

sudo apt install unicode # unicode 你好， 转换为对应uniode编码

sudo apt install axel # 基于字符界面的下载工具（比wget与curl强大）
# 支持断点续传等等



```



### 安装java



```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer oracle-java8-set-default
source /etc/profile

# 将oracle的java设为默认环境，替换open java

```





### 中文输入法

```
sudo apt install ibus ibus-pinyin

imconfig  # 运行，安装向导，选择ibus

reboot

设置，将只能中文输入法加入（或者supingyin）

# ibus有个小bug，手动修复：
ibus-setup
然后取消上面的勾选




```





### 常用软件

- chrome
- Golddict   （中英文翻译）
- 
- 
- 



##### 邮件客户端

- mailspring          # UI美观，但是性能不是很好
- thunderbird      # 性能好，太丑
- Protonmail Desktop   # **最安全的邮件**（质子形状的） **推荐安全从业者**



##### Office

- WPS for Linux    # 唯一推荐
- onlyoffice       # 客户端不成熟，可以集成到云上，



##### to-do-list           （列表清单软件）

zenkit      # 打开ubuntu的软件仓库，可以直接搜索安装 ， 也可以做为看板类型软件





##### 印象笔记

Tusk   # 软件仓库搜索安装即可，可以同步印象笔记



##### 书籍管理

calibre



##### 虚拟机

virtualBox / VMWare Workstation



##### 下载工具

- uGet     # 类似图形化的axel，支持断点续传
- aMule
- Transmission



##### 图形图像视频编辑

- GIMP    # 开源软件中的ps
- VLC       
- kenlive   # 视频编辑
- pitivi      # 视频编辑



##### 思维导图

MindMaster         # 国产



##### appimage 程序下载

**打包软件方式，大大降低了软件安装的成本（类似于绿色软件方式），建议多逛逛**

https://appimage.github.io







---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ubuntu%E6%8E%A8%E8%8D%90%E7%94%9F%E4%BA%A7%E5%8A%9B%E8%BD%AF%E4%BB%B6/  

