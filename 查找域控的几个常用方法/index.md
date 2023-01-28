# 查找域控的几个常用方法



  
#### 查找域控的几个常用方法
  
1.net view

```powershell
net view /domain
```

2.set log

```powershell
set log
```

3.通过srv记录

```powershell
nslookup -type=SRV _ldap._tcp.corp
```

4.使用nltest

```powershell
nltest /dclist:corp
```

5.使用dsquery

```powershell
DsQuery Server -domain corp
```

6.使用netdom

```powershell
netdom query pdc
```

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E6%9F%A5%E6%89%BE%E5%9F%9F%E6%8E%A7%E7%9A%84%E5%87%A0%E4%B8%AA%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95/  

