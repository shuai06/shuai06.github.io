# Manjaro 安装之后 基本配置



  
# Manjaro 安装之后 基本配置



**配置国内源**

```bash
sudo pacman-mirrors -i -c China -m rank   #弹出来框，我选择的清华的源


```

**升级系统**

```bash
sudo pacman -Syy && sudo pacam -Syyu
```



**安装vim**

```bash
sudo pacman -S vim
```

**添加arch源**

```bash
sudo vim /etc/pacman.conf
#在文件末尾添加，我添加的清华的

```

**安装archlinuxcn签名钥匙**(导入 GPG key，否则的话key验证失败会导致无法安装软件)

```bash
sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring
```



**安装网络工具包，ifconfig等**

```bash

pacman -S net-tools dnsutils inetutils iproute2
```



---





**安装sogou拼音输入法**

```bash
sudo pacman -S fcitx-im 	# 安装fcitx 选择全部安装
sudo pacman -S fcitx-configtool 	# fcitx 配置界面
sudo pacman -S fcitx-sogoupinyin 	# 安装sogoupinyin
```

**设置中文输入法环境变量**，编辑~/.xprofile文件，增加下面几行(如果文件不存在，则新建)    重启后生效

```bash
sudo vim ~/.xprofile # 打开编辑.xprofile文件
# 在文件中加入以下两行代码
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```



**安装Google-Chrome浏览器**

```
sudo pacman -S google-chrome
```

**安装网易云音乐**

```
sudo pacman -S netease-cloud-music
```

**安装git (默认貌似就已经存在了)**

```
sudo pacman -S git
```

**安装wps，及其字体**

```
$ sudo pacman -S wps-office     # 安装wps
$ sudo pacman -S ttf-wps-fonts  # 安装wps字体
```

**配置wps，使wps可以输入中文**

```
$ sudo vim /usr/bin/wps     # 编辑wps配置文件
# 在 紧跟#!/bin/bash后添加下列三行
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

注销后再次登录输入法生效。



**安装Shadowsocks-qt5，实现科学上网**（我还是一会安ssr吧，在文章最底下）

```
sudo pacman -S shadowsocks-qt5
```

**安装VSCode**

```
 sudo pacman -S visual-studio-code-bin  # 安装VSCode和云音乐等等，都需archlinuxcn源
```

**安装markdown编辑器**

```
sudo pacman -S typora
```

**配置jdk环境变量**

```
# 配置环境变量
# sudo vim /etc/profile #编辑文件
# 在文件末尾处追加下列几行
export JAVA_HOME=你的jdk解压后的绝对路径 
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export  PATH=${JAVA_HOME}/bin:$PATH
```

**安装终端显示信息neofetch(装b神器,安装screenfetch也可)**

```
sudo pacman -S neofetch
```

**安装Tim,微信 (KDE桌面无法使用)**

```
sudo pacman -S deepin.com.qq.office deepin.com.wechat
```

**配置微信，tim的中文输入问题 (KDE桌面无法使用)**

Tim启动脚本位置：/opt/deepinwine/apps/Deepin-TIM/run.sh WeChat启动脚本位置：/opt/deepinwine/apps/Deepin-WeChat/run.sh # 1 配置tim中文，打开tim启动脚本文件 （微信同理）

```
sudo vim /opt/deepinwine/apps/Deepin-TIM/run.sh
```

在启动脚本命令之前添加以下内容

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

**安装Deepin Terminal(终端)**，安装完成之后，自行在deepin-terminal更改显示字体

```
sudo pacman -S deepin-terminal
```

**安装有道词典**

```
sudo pacman -S youdao-dict
```

**安装vokoscreen录屏**

```
 sudo pacman -S vokoscreen
```

**安装MariaDb代替mysql(MyriaDb与Mysql相互兼容)**

```
$ sudo pacman -S mariadb mariadb-clients
# 安装成功后，根据提示，输入下列指令初始化MariaDb数据库
$ sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
# 一番信息自动输出完成后，执行以下代码
$ sudo systemctl start mysqld # 启动MariaDb
$ mysqladmin -u root password "root" # 为root、用户添加密码
$ sudo systemctl enable mysqld # 设置mariaDb开机自启
$ mysql -uroot -p # 输入设置的的密码，登录数据库
```





**安装配置oh my zsh**(听说是bash的升级版，装*专用，我就先不弄了)

```
$ sudo pacman -S zsh
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" # 下载并配置ohmyzsh
$ chsh -s /bin/zsh  #更换默认bash，重启后生效
```



**卸载Manjaro自带的一些没用的软件**



卸载之后就可以更新一下软件了

```
sudo pacman -Syyu
```



---

## pacman基本命令



**更新软件**

```bash

sudo pacman -Su		# 更新已安装软件

sudo pacman -Syu	#更新软件前检查包列表是否最新

sudo pacman -Syyu     #更新前强制下载包列表             
```



**查找软件**

```
sudo pacman -Ss packge_name
```





**安装指定的包**

```
 pacman -S package_name1 package_name2 ...
```



**安装包组**

一些包属于一个可以同时安装的**软件包组**，例如：

```
 pacman -S gnome
```



想要查看哪些包属于 gnome 组，运行：

```bash
pacman -Sg gnome
```





**删除软件包**

1.删除单个软件包，**保留**其全部已经安装的依赖关系

```bash
pacman -R package_name
```

2.除指定软件包，及其所有**没有被其他已安装软件包使用的依赖**关系：

```bash
pacman -Rs package_name
```

3.要删除软件包和**所有依赖**这个软件包的程序

```
pacman -Rns package_name
```

4.要删除软件包，但是**不删除依赖**这个软件包的其他程序：

```bash
pacman -Rdd package_name
```

5.pacman 删除某些程序时会备份重要配置文件，在其后面加上*.pacsave扩展名。-n 选项可以避免备份这些文件：

```bash
pacman -Rn package_name1
```

​		**注意: pacman 不会删除软件自己创建的文件(例如主目录中的 .dot 文件不会被删除。**





**清理软件包缓存**

pacman 将下载的软件包保存在 **/var/cache/pacman/pkg/** 并且不会自动移除旧的和未安装版本的软件包，因此需要手动清理，以免该文件夹过于庞大。

使用内建选项即可清除未安装软件包的缓存：

```bash
pacman -Sc
```

**警告**:仅在确定当前安装的软件包足够稳定且不需要降级时才执行清理。

**pacman -Sc**仅会保留软件包的当前有效版本，旧版本的软件包被清理后，只能从其他地方如 Arch Linux Archive (简体中文)中获取了。

**pacman -Scc** 可以清理所有缓存，但这样 pacman 在重装软件包时就只能重新下载了。除非空间不足，否则不应这么做。





#### 更新源

**自动：**
自动设置最快的源

```
sudo pacman-mirrors -g
sudo pacman -Syyu
```



**手动：**
选择源：`sudo pacman-mirrors -i`
同样要强制更新包

重置为自动选择：

```
sudo pacman-mirrors -g -c all
```

切换testing分支

```
sudo pacman-mirrors -g -b testing
sudo pacman -Syyu
```

退回稳定版

```
sudo pacman-mirrors -g -b stable
sudo pacman -Syyuu
sudo pacman-mirrors -i -c China -m rank
```



---

### 清理垃圾

清除系统中无用的包

```
sudo pacman -R $(pacman -Qdtq)
```



清除已下载的安装包

```
sudo pacman -Scc
```




日志垃圾
查看日志文件

```
du -t 100M /var
```

或

```
journalctl --disk-usage
```




删除指定大小日志文件

```
sudo journalctl --vacuum-size=50M
```



------

### 安装ssr

**步骤：**

1. 安装git命令，用

   ```
   git clone -b manyuser https://github.com/shadowsocksrr/shadowsocksr.git
   ```

   下载shadowsockssr。

2. 进入shadowshocksr目录  运行如下命令：
   `./initcfg.sh`

3. 运行如下命令编写user-config.json配置文件
   `sudo nano user-config.json`                  将ssr服务器的配置填写到配置文件中

4. 进入shadowsocks目录
   `cd shadowsocks`
   `./local.py   `            代理客户端已经开始运行

5. 设置浏览器代理：
   以firefox为例， 从 https://addons.mozilla.org/zh-CN/firefox/addon/foxyproxy-standard/?src=search 添加foxyproxy-standard插件
   左键点击插件图标 选 “options” – “add” –  “socks5”
   填写代理的名称（随便写），IP（127.0.0.1），端口（1080）

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/manjaro-%E5%AE%89%E8%A3%85%E4%B9%8B%E5%90%8E-%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE/  

