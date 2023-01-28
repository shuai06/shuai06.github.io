# ArcEngine"万能""加载数据方法



  
**下面介绍一种“万能”加载数据的办法：** 

```c#
// 加载很多种数据
ICommand command = new ControlsAddDataCommandClass();
command.OnCreate(mainMapControl.Object);
command.OnClick();
mainMapControl.ActiveView.Refresh();
```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/arcgisengine-%E4%B8%87%E8%83%BD%E5%8A%A0%E8%BD%BD%E6%95%B0%E6%8D%AE%E6%96%B9%E6%B3%95/  

