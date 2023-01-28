# ArcEngine同步数据视图与布局视图


  

```C#
// 数据视图与布局视图同步
private void mainMapControl_OnAfterScreenDraw(object sender, IMapControlEvents2_OnAfterScreenDrawEvent e)
{
    IActiveView pActiveView = (IActiveView)mainPageLayoutControl1.ActiveView.FocusMap;
    IDisplayTransformation displayTransformation = pActiveView.ScreenDisplay.DisplayTransformation;
    displayTransformation.VisibleBounds = mainMapControl.Extent;
    mainPageLayoutControl1.ActiveView.Refresh();
    CopyToPageLayout();  // 调用下面的函数

}

```


```c#
// CopyToPageLayout() 布局视图与数据视图同步
private void CopyToPageLayout()
{
    IObjectCopy pObjectCopy = new ObjectCopyClass();
    object copyFromMap = mainMapControl.Map;
    object copiedMap = pObjectCopy.Copy(copyFromMap);   // 复制地图到copiedMap中
    object copyToMap = mainPageLayoutControl1.ActiveView.FocusMap;
    pObjectCopy.Overwrite(copiedMap, ref copyToMap);   // 复制地图
    mainPageLayoutControl1.ActiveView.Refresh();

}
```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/arcgisengine-%E6%95%B0%E6%8D%AE%E8%A7%86%E5%9B%BE%E4%B8%8E%E5%B8%83%E5%B1%80%E8%A7%86%E5%9B%BE%E7%9A%84%E5%90%8C%E6%AD%A5/  

