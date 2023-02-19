# ubuntu server 包管理




<img src="https://image.geoer.cn/20200126205055895_1655667133.png" />

<img src="https://image.geoer.cn/20200126210117946_1080742536.png" />

<img src="https://image.geoer.cn/20200126210349356_2002672330.png" />


#### DPKG包管理器（本地）
<img src="https://image.geoer.cn/20200126210801883_1750439429.png" />

​     

`dpkg -l` 导出软件包，在新系统上安装   
`dpkg -L xxd`  查看某个软件包在电脑上包含哪些文件  
`dpkg -S /usr/.../man1`  看这个文件来自哪个软件包  
`dpkg -i xxxname_version_架构.deb` 安装      // 架构(amd64, i386)
`dpkg --print-architecture`  查看系统的支持的架构 
`dpkg --print-foreign-architectures`   查看系统是否支持其他的架构    

`cat /var/lib/dpkg/arch`   查看系统是否可以拓展支持其他架构类型
 `  `  可以让系统支持其他类型架构
`dpkg --remove-architecture i386`   删除支持某个架构

`dpkg -r name:amd64`  卸载软件包（可以带上架构） 不删除配置文件
`dpkg -P name`   卸载并删除配置





#### APT包管理器  


```bash
# 源文件
cat /etc/apt/sources.list | grep -v ^#   不显示#号开头的  
/etc/apt/sources.list.d/ 目录下面是第三方的软件源 


sudo apt upgrade   # 对已经安装的包更新

sudo apt dist-upgrade  # 更新新包，删除旧包（包含内核）

sudo apt remove nmap --purge   # 删除包和配置文件 
-- /var/log/dpkg.log  

apt list  --upgradeable  # 查看可以更新的包

apt serach 'network mapper'  # 搜索

sudo apt show nmap  # 查看软件包详细描述  

# 卸载, 卸载完成之后再把不用的autoremove等
sudo apt remove nmap --purge
sudo apt purge nmap

sudo apt autoremove   # 谨慎使用！！！（某系文件可能是旧版内核的依赖包，如果回退老版本可能会出现问题）

/var/cache/apt/archives   # 下载到这里来了（里面不需要的deb包可以删除。只是下次再次需要的时候重新下载）
/var/lib/apt/   # 更新源的索引文件.list

# .deb 是打包好的，不显示源代码
apt download nmap   # 下载但是不安装(.deb包)   
apt source asw # 把源代码下载下来，可以可以下载之后自己编译安装  
apt showsrc nmap  # 查看源代码  


```



#### 自动更新（无人值守）  
```bash
sudo apt install unattended-upgrades
# 配置主配置文件
/etc/apt/apt.conf.d/50unattended-upgrades
允许：一般只允许security那个  
黑名单：
# 配置文件2
/etc/apt/apt.conf.d/10periodic
更新/下载/清除/安装 周期

# 重启服务
sudo service unattended-upgrades restart
sudo systemctl restart unattended-upgrades.service  # 新版本ubuntu推荐


# 日志
cat /var/log/unattended-upgrades/unattended-upgrades-xxxxxxxx.log

sudo lsb_release -a  # 查看版本和code_name
```


###### 无人值守更新通知
```bash
# 配置文件 /etc/apt/apt.conf.d/50unattended-upgrades中的mail，指定邮箱

# apticron软件包，用来发邮件的
sudo pt install apticron



```



> 多手动练习，少复制






## APT更新源配置
**推荐使用官方更新源**  

<img src="https://image.geoer.cn/20200127152919657_1859338354.png" />

deb   指的是安装包文件  
deb-src  是deb相关的，还没有编译成deb的源文件  

生产环境中，尽量用前两种类型的  

**第三方库  **

<img src="https://image.geoer.cn/20200127153629331_1188888659.png" />

`apt-key add`
`apt-ket del`


###### PPA
<img src="https://image.geoer.cn/20200127154253164_223311205.png" />



###### SNAP包管理 
> 发展趋势非常好

<img src="https://image.geoer.cn/20200127155257588_792061499.png" />


###### SNAP包管理

可以先安装snap: `apt install snap`  

```bash
sudo snap find nmap

sudo snap install nmap

sudo snap remove sudo snap

sudo snap refesh nmap   # 更新单个软件包

sudo snap refresh  # 更新索引文件

# 同时安装多个软件包（彼此独立）  
# 作为apt的补充

# snap的软件包会下载到一个目录： /snap/bin/...

```











---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E5%8C%85%E7%AE%A1%E7%90%86/  

