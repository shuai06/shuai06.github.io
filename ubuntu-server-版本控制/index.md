# ubuntu server-版本控制


  
# 107-版本控制

## 概述
#### 简介
version control system（版本控制系统）
revision control system (修订控制系统)

- 开发工具，但不仅限于源码文件
- 记录文件的每次变更
- 随时可以回到历史状态
- 文件以及元数据的变更（修改人、修改理由、具体修改内容、时间戳）
- 配置文件是除数据之外第二大资产
- 团队写作


#### 分类
- Centralized version control systems (SVCS) 集中式版本控制
    - 集中存储修改，颗粒控制细，但是单点故障隐患
    - CSV / Subersion

- distributed  version control systems (DVCS) 分布式版本控制
    - 分布式存储库的完整拷贝
    - Git / Mercurial  / Bazaar(Ubuntu官方在开发时候使用的)
    - 每个人维护本地库，完成后上传到服务器

指针指向当前最新版本
随着回滚还原历史版本



## Git
- 最初由Linux内核创始人开发
- Git作为开发工具而生
- 适合大型项目的版本管理
- Git库保存所有程序员的团队成果
- Git默认使用OpenSSH
- 服务器上的Git用户需要能够修改目录(库)
- 一个库就是一个文件夹(库配置信息: `.git`目录)
- 无需独立的专职服务器，满足存储空间需求可共用
- github目前世界上最大的代码托管服务商（已被微软收购）



#### 服务端
**安装**
```bash
# 1. ubuntu自带的(和git自带的版本不一样,不是最新的)
# sudo -s    把自己提升成root权限，后面就不用输入sudo了
apt install git 

mkdir /git    # 创建一个文件夹(后续作为一些git库的存放位置)
chown xps:xps /git
cd /git
git init --bare apache2    # 创建一个裸库,名字叫apache2（空的框架库，不存在数据）

/var/log/auth.log    # ssh登录问题诊断



# 2.使用ppa库方式安装
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git
git version    # 版本比ubuntu库中的新

然后和上面一样创建存放git库的目录
```


#### 客户端
使用
```bash
# 客户端和服务端都是安装的同一个git软件包
# 其实客户端本身也可以作为一个本地的git服务器

# 基本配置
git config --global user.name "Your Name"
git config --global user.email "eamail@domain.com"
git config --list    # 查看配置信息

git help    # 查看常用的git二级命令
git help -a    # 查看所有的子命令
git help -g    # 查看所有的子命令, 并有描述
git help everyday    # man giteveryday  查看具体某一个子命令的用法

```

git客户端连接服务器
```bash
mkdir /git    # 同样创建一个文件夹
chown xps:xps /git
cd /git

# 作为客户端，从远程服务端clone库(是服务端的绝对路径)
git clone xps@192.168.0.105:/git/web


# .git目录存放配置信息(使之成为git库)，删除此目录就是一个普通文件夹
# 其中config文件包含远程服务器地址信息(写好之后以后就不用手动输入信息；)
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = xps@192.168.0.105:/git/web
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master

```


#### 常用git命令
```bash
# 查看状态
git status

# 追加变更到git的追踪
git add .    # 所有文件
git add file_name    # 指定文件

# 提交变更给本地git库
git commit -a -m "第一次提交"

# 同步到服务器（orign等在config文件已经写好了）
# 提交之前必须服务器的内容是最新的版本，先git pull, 再push
git push orign master

# 从服务器更新
git pull    # 服务端数据存储结构与客户端不同(数据库)

# 撤销文件的变更(不仅仅对文件，对版本也可撤销)
git checkout <file>    # 回退最近的版本

# 比对来查看文件的修改变化
git diff index.html    
# 内容(对比的两个文件)
--- a/4.conf
+++ b/4.conf


########### 切回历史上任何一个版本
apt install tig    # 方法很多，这里使用了tig
tig    # 直接查看历史版本(移动到版本上，下面会有版本号hash值)，回车会看到具体变更内容

git checkout ddwe23e353c0d6665406e7c07djaoid2d   # 切回到指定版本
# 只是指针临时指向了之前的版本，没显示后面的版本(但是后面的版本还是存在的)

# 如果验证没问题，先回到最新状态
git checkout master

# 回滚(只是回滚那一个版本的所做的变更, 不是回退到那，也不影响之后的版本)  --> 从当前最新版本的基础山，去掉前面某一个版本中所做的变更
git revert master ddwe23e353c0d6665406e7c07djaoid2d


```

**不仅仅的对代码，对配置文件也可以**
是个好习惯
```bash
# 例子 :apache的配置文件
cp -rp /etc/apache2 /etc/apache2.bak
mv /etc/apache2/* /git/apache2
rm /etc/apache2    # 这里千万不能重启
find /git/apache2 -name '.?*' -prune -o -exec chown root:root {} + # 排除.git(.?*)，其他的都改为root
ln -s /git/apache2 /etc/apache2    # 做个软链接(可以让应用读取到配置文件)
systemctl reload apache2

# 以后修改就在git仓库修改，并进行add和commit和push，以便出问题回滚
```




#### Github
>开源项目可以使用，前面自己在服务器搭建的git服务器可以自己公司用(商用的肯定要自己内部建版本控制服务器)
>机密&隐私信息不建议放到开源平台


**创建并同步新库**
```bash
## 1.在本地新建(已有库)，然后关联到github仓库
echo "# new" >> README.md
git init
git add README.md
git commit -m "first"
git remote add origin https://github.com/xps/new.git    # 连接远程仓库，同步现有库
git pull    # 先把服务器的版本同步到本地，然后才能正常提交
git push -u origin master    # 提交到远程github仓库

# 如果要用ssh协议， 要在github的仓库的setting的的Deploy Key
# 然后本地 .ssd/id_rsa.pub 公钥粘贴上去
# 然后就不用输入密码了


## 2.在github新建，然后clone下来，再提交
```


#### GitHub pages
通过github上传页面文件建站(目前只是静态的)
```bash
# 新建库名为： user_name.github.io
# 现在本地创建已有库也行(参照同步到github库)
git clone https://github.com/user_name.github.io
cd user_name.github.io/
echo "hello world" > index.html    # 可以在本地设置好页面(静态网站)，再上传
git add .
git commit -m "init commit"
git remote add origin https://github.com/user_name.github.io
git push -u origin master

# 浏览器访问 https://user_name.github.io
```


#### SSH Key连接库
```bash
Clonr or Download ->    CLone with SSH
Setting ->    Deploy keys    -> Add
git clone git @github.com:xps/test.git
```



#### GitLab
GitLab是开源的Git库Web界面（包含团队写协作工具 GitLab mattermost ->即时聊天沟通的）
基于Ruby语言开发的Web应用程序
所有组件全部打包集成（Postgresql, redis, nginx, logrotate, Sidekiq, Unicron）
如果想要要支持MySQL数据库，社区版需要手动安装
安装需求
```bash
最小：单核，1GB，适当存储空间(400MB)，100用户以下(建议双核、2GB以上)
```

**安装**
```bash
### 集成安装方式
https://packages.gitlab.com/gitlab/gitlab-ce/
# ubuntu(安装软件源)
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
# 开始安装(等待时间比较长)
sudo apt install gitlab-ce

# 自动初始化配置
sudo gitlab-ctl reconfigure

# 查看状态
sudo gitlab-ctl status

# 然后就可以访问了， 设置初始root帐号密码
# 默认是只有一个root管理员
# 和github等的使用体验是类似的

```


**配置**
```bash
/etc/gitlab/gitlab.rb   # 配置文件

external_url    # 指定访问域名(也可以启用https)
letsencrypt 也集成了  # 免费https
database    # 指定外部PosgreSQL数据库
email    # 设置出站邮件服务器(注册帐号链接等)


sudo gitlab-ctl reconfigure        # 每次修改后都要重新配置
fitlab-ctl show-config    # 显示已经启用的配置
sudo gitlab-ctl status
```


**常用命令**
```bash
sudo gitlab-ctl status   # 查看状态
sudo gitlab-ctl stop     # 停止gitlab
sudo gitlab-ctl start    # 启动gitlab
sudo gitlab-ctl restart  # 重启gitlab

sudo gitlab-ctl  tail    # 查看所有日志
sudo gitlab-ctl tail nginx/gitlab_access.log    # 查看nginx访问日志
sudo gitlab-ctl tail postgresql    # 查看postgresql访问日志
```


**新建用户帐号**
F1: 管理员登录 -> 扳手图标(全局设置) -> 用户
```bash
普通用户的创建库的限制一般设置为0

创建帐号的时候不能设置密码(是发送邮件他自己设置)，但是创建完成之后再次点击用户就可以修改密码了
```

F2: 默认是支持普通用户自助注册的(不建议，因为一般是在企业内部使用)
```bash
# 限制/禁用普通用户自助注册
管理员登录 -> 扳手图标(全局设置) -> Setting
Sign-up Restrictions(注册限制)  -> Sign-up enable 不选中   # 禁用自助；只允许手动创建帐号
Send confirmation email on sign-up    -> 指定邮箱域名白名单(*.lab.com)
Doamin Blacklist -> 设置黑名单
```


**建库**
设置(私有,内部,公开) -> 选择库成员(设置-->成员->设置用户权限和期限)，clone，同步
默认是http协议访问，可以改为ssh公钥访问



一定程度上支持中文(settings -> Perferences -> Language)


普通用户第一次登录也要重新设置密码，
就可以在本地clone项目库，
然后就可以同步了(其他步骤同github)


**用户设置 -- Profile**
```bash
邮件，名称(给别人看，方式穷举暴力破解)
# 用户资料
full name 修改
email 修改, 且不公开
```

**帐号名称**
```bash
Account -> root修改为别的  # 安全
# 帐号(路径这里才是)
修改得隐蔽一点
```


**SSH Key**
```bash
ssh-keygen    # 现在本机生成公钥
~/.ssh/id_rsa.pub    # 生成的公钥
Add Key        # 拷贝公钥到gitlab的key的地方
```


**禁止普通用户建库**
```bash
设置用户可创建0个库(Project)

Accoumt and Limit Settings -> Default projects limit -> 0
```


**Let's Encrypt证书更新**
```bash
# 自动更新时间
vim /etc/gitlab/gitlab.rb
letsencrypt['auto_renew'] = true    # 开启自动更新
letsencrypt['auto_renew_hour'] = "12"  # 每12个小时
letsencrypt['auto_renew_minute'] = "30" # 30分钟
letsencrypt['auto_renew_day_of_month'] = "*/7"  # 每隔7天
```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/ubuntu-server-%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6/  

