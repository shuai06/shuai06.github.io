# ubuntu server-虚拟化


## <font color=red> 云计算 </font>
>资源就像水龙头里的水，按需索取


**信息时代计算资源是最主要的生产力**
- 主机时代、网络时代
- 集中和分散、三十年河东三十年河西
- 量子计算可能颠覆一切


**计算资源的云化**
- 按需调度、按需索取
- 计算资源需要更大程度的颗粒度细化
- `虚拟化`是计算资源细分的`基础`


**处理器运算能力的大幅提升**
- 处理速度月来越快（摩尔定律）
- 多核处理器架构
- 大量计算资源限制浪费，企业整体运行成本高
- 多应用同机部署，应用隔离差，稳定性影响大
- 处理器支持虚拟化技术


###### 分类
**1.Saas：** Software as a Service
```bash
# 软件即服务， 服务以软件形式交付
Gmail / Google Docs / 在线Office / Office 365 ...
无需本地安装部署
```

**2.PaaS：** Platform as a Service
```bash
# 平台即服务
提供软件开发运行的基础平台
API、Google App Engine
```

**3.IaaS：** Infrastructure as a Service
```bash
# 基础架构即服务
最底层的晕计算，物理的计算、存储、网络资源饭各位你
AWS、Google Computer Engine、 阿里云
```


云计算是将资源打碎、分配、发布、交付的一组工具的合集

Hypervisors -> 使得虚拟服务器的性能不依赖物理机，和物理机是平行的，可以直接访问物理的CPU
```bash
# 常见的虚拟化软件
KVM、XEN(目前部署方式是完整的光盘)、QEMU、Virtualbox、VMWare
# 本章主要学这两个：KVM、EMU
```

云计算平台
```bash
# ubuntu上最典型的
Openstack

```
服务编排工具
```
# 典型的
Juju
```

机器配置工具
```
MAAS
```


## <font color=red> 虚拟化 </font>
物理服务器，部署虚拟化平台，创建多个虚拟机，每个虚拟机使用物理机的计算量。虚拟化是云计算的基础

- 降低成本，提高利用率，方便管理部署，快速恢复还原
- 普遍用于开发测试实验、生产环境部署都可
- 需要硬件支持(BIOS设置开启)
- CPU特性 Hypervision


### KVM
**Kernel-based Virtual Machine（KVM）**

- 基于内核的虚拟机(KVM)和快速仿真器(QEMU)组成的虚拟化套件( QEMU/KVM )
- QEMU/KVM在没有硬件虚拟化支持下，虚拟机需要运行在QEMU模拟器中
- 有硬件虚拟化支持下，**接近硬件物理机速度运行**
- 虚拟化扩展决定性能( virtualization extensions )
- Ubuntu默认内建支持的虚拟化方案
- 内建于Linux内核( Intel 的kvm-intel.ko或AMD的kvm-amd.ko )
- libvirt库是Linux与虚拟化通信的统一 接口
- 处理在主机和客户机之间分离任务所需的低级指令
- **无法并行运行VirtualBox和KVM虛拟机**，它们会抢占CPU的虚拟化功能

把`KVM`和`QEMU`结合在一起
通过`libvirt`库实现Linux与虚拟化通信的接口



**验证硬件平台支持虚拟化拓展(较新的硬件全部支持)**
四种方法
```bash
# F1
egrep -c '(vmx|svm)' /proc/cpuinfo  # 结果不为0即支持
erep -E 'vmx|svm' /proc/cpuinfo  # 结果不为0即支持


# F2 
kvm-ok
# sudo apt install cpu-checker


# F3
lscpu
# 找下列字段：
# Virtualization: VT-x
# Flags: vmx


# F4  
sudo virt-host-validate
# sudo apt install libvirt-client
# Checking for hardware virtualizatin: PASS

可以在virtualbox或vmware虚拟机里面安装kvm的虚拟机
```


#### KVM安装和使用
##### 安装服务端组件
```bash
# bridge-utils桥接的防止，用来做地址转换的
sudo apt install bridge-utils libvirt-bin qemu-kvm qemu-system
# 当前用户会被加入livirt组
# 需要重启系统使得权限生效


# 映像文件目录(以后的虚拟机文件都会放在这里)
/var/lib/libcirt/images
# 配置文件
/etc/libvirt/libvirtd.conf
权限(读写,管理)的修改，加密证书等。一般不需要配置。
```

##### 安装客户端组件
###### 1.图形化的客户端管理工具
```bash
# 1.图形化的管理工具(virt-manager)，ssh-askpass是用来ssh连接远程主机使用的
sudo apt install virt-manager ssh-askpass 

# 客户端的使用
可以新建连接：选择连接哪个机器(本机、远程机器)的kvm后台
创建虚拟机:跟virtualbox的就类似了，Add Pool指定iso文件的路径来挂载寻找


# 2.也有字符界面的客户端管理工具
```



**网卡桥接配置**
默认虚拟机网络模式为NAT（可以访问外部网络，但外部计算机无法访问虚拟机），可以使用`网桥`实现VM被外部网络访问

<img src="http://image.xpshuai.cn/20200510211152045_955020812.png" />

网桥的配置
```bash
# sudo vim /etc/netplan/50-cloud-init.yaml
# 新增配置
ethernets:
    ens33:
        dhcp4: no    # 这个分给网桥，所以这个物理网卡不能有ip地址
        dhcp6: no
    bridegs:    # 配置网桥
        br0:
          interfaces: [ens33]
          dhcp4: no
          addresses: [172.16.24.13/24]
          gatewat4: 172.16.24.1
          nameservers:
            addresses: [8.8.8.8,8.8.4.4]
    

# 让配置生效
sudo netplan apply
```

虚拟机配置：选择虚拟机，选择第二个窗口，选择网卡，Bridge模式，选择设备，启动虚拟机


建立连接
```bash
virt-manager 本机/ 网络 连接
virt-manager 基本使用
```

克隆创建虚拟机
```bash
KVM不支持模板功能，但可将虚拟机关机作为模板使用(右键，clone, 基于它作为"模板")
# 创建虚拟机快，占资源少
```


虚拟机管理控制台（有很多好用的功能）


**迁移(Migrate) **--> 把某宿主机上(出现故障了)某台虚拟机迁移到另一台安装了kvm服务的服务器上



###### 2.命令行客户端
> 命令行方便写脚本，·批量进行很多虚拟机的安装和管理

创建虚拟机
```bash
# 命令行创建虚拟机 virt-install
sudo virt-install -n web1 -r 2048 --disk path=/var/lib/libcire/images/web1.img,bus=virtio,size=10 -c ubuntu-18.04-server-amd64.iso --newwork newtork=default,model=virtio --graphics vnc, listen=0.0.0.0 --noautoconsele -v

# --disk path   虚拟机镜像文件存放路径
# size=4        硬盘文件大小4G
# bus=virtio    总线类型
# -c            CDROM（iso文件、物理光驱）
# --newwork newtork=default，model=virtio   接口模式
# --graphics vnc, listen=0.0.0.0        VNC控制接口
# --noautoconsele    不自动连接虚拟机控制台
```

复制虚拟机
```bash
sudo virt-clone -o win10 -n win10-2 -f /var/lib/libcire/images/win10-2.img
# -o    原始虚拟机
# -n    新建虚拟机
# -f    虚拟机镜像文件存放路径
```


建立连接
```bash
virsh connect qemu:///192.168.0.12
```

查看vm服务器上所有VM
```bash
#  查看run的
virsh list

# 查看所有状态的
virsh list --all
```

管理VM
```bash
virsh console centos_01
```

关闭当前console
```bash
ctrl + ]
```

正常关机
```bash
virsh shutdown centos_01
```

强制关机
```bash
virsh destory centos_01
```

删除虚拟机
```bash
virsh undefine centos_01
```

挂起VM
```bash
virsh suspend centos_01
```

恢复使用VM
```bash
virsh resume centos_01
```





#### 使用云平台

下载适合不同环境的镜像: https://cloud-images.ubuntu.com/releases/bionic/release
```bash
wget /ubuntu-18.04-server-cloudimg-amd64.img  -O u18.img.dist
```

解压下载的镜像文件
```bash
qemu-img convert -O qcow2 u18.img.dist u18.img

# 不能直接使用，可以理解这个为一个模板
```

创建用户数据文件
```bash
vim cloud-config

# cloud-config
passwdor: 123465
chpasswd: { expire: Flase }  # 密码不过期
ssh_pwauth: True    # 允许ssh身份认证
```

生成用户数据盘
```bash
# 基于模板创建一个新的数据盘
cloud-localds user-data.img cloud-config  # 使用上面的配置文件
```

启动虚拟机
```bash
kvm -m 2048 -smp 2 -hda u18.img -hdb user-data.img -net nic -net user,hostfwd=tcp::1810-:22 -nographic

# 帐号密码: ubuntu / 123465(上面配置文件指定的)
sudo passwd ubuntu     # 修改密码
```

删除云镜像初始化脚本
```bash
sudo apt-get remove cloud-init
```

使用云镜像创建VM


部署完一个之后，就可以基于第一个进行clone



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/ubuntu-server-%E8%99%9A%E6%8B%9F%E5%8C%96/  

