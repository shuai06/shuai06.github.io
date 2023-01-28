# 一条命令修改windows注册表


> 在网上看到大佬们的文章，我也在此记录以下



这里以修改windows远程桌面端口3389为33333为例子。

修改注册表，命令如下：

```shell
REG ADD “HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp” /v “PortNumber” /t REG_DWORD /d 3333 /f
```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E4%B8%80%E6%9D%A1%E5%91%BD%E4%BB%A4%E4%BF%AE%E6%94%B9windows%E6%B3%A8%E5%86%8C%E8%A1%A8/  

