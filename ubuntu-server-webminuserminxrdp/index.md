# ubuntu server-远程管理 Webmin&usermin&Xrdp




## Webmin

- 基于WEB的系统远程管理
- 基于图形化的友好操作界面
- 多开启一个服务端口，增加了攻击面
- Webmin
- WEB应用程序，完成部分服务器日常管理操作



###### 安装依赖包

```bash
apt -y install python apt-show-versions libapt-pkg-perl libauthen-pam-perl libio-pty-perl libnet-ssleay-perl
```



###### 下载安装

```bash
# 下载安装
curl -L -O htt://www.webmin.com/download/deb/webmin-current.deb
dpkg -i webmin-current.deb	# 安装

# 配置允许访问的ip（这里配置本地地址 和 某个网段）
vi /etc/webmin/miniserv.conf
	allow=127.0.0.1 10.0.0.0/24

# 重启服务
systemctl restart webmin

```

###### 访问

```bash
# 访问
https://10.1.1.1:10000
# 使用系统账号登录，建议root


## 管理系统
如果多台服务器，可以用"广播服务器"来寻找其他装了Webmin的服务器，在一个web端来统一管理

```





## usermin

普通用户自服务`usermin`



**安装依赖包**

```
apt -y install python apt- -show-versions libapt-pkg-perl libauthen-pam-perl libio-pty-perl libnet-ssleay-perl
```

**安装**

```bash
curl -L -0 http:/ /www.webmin.com/download/ deb/usermin-current.deb
dpkg -i usermin-current.deb

vi /etc/usermin/ miniserv.conf
  allow=127.0.0.1 10.0.0.0/24 
  denyuser=root	# 禁止root登录usermin
  
systemctl restart usermin
```



**访问**

```bash
https://10.0.1.12:20000

# 
```



## Xrdp远程桌面

- VNC vs XRDP

- 使用Windows远程桌面连接工具远程管理Linux Server

- 图形化桌面作为后段

- Gnome、 KDE、Xfce (越来越火)

  

**安装Xfce桌面环境**

  ```bash
# 以Xfce为例
sudo apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils
  ```

**安装XRDP**

  ```bash
sudo apt install xrdp
  ```

**配置**

```bash
sudo vi /etc/xrdp/xrdp.ini
	exec startxfce4	# 这里是xfce4
```

**重启服务**

```bash
sudo systemctl restart xrdp
```

**防火墙**

```bash
sudo ufw allow 3389

sudo ufw allow from 10.0.0.0/24 to any port 3389	# 可以限制来源ip地址
```

**连接**

用Windows的远程连接工具(mstsc)连接Linux服务器


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ubuntu-server-webminuserminxrdp/  

