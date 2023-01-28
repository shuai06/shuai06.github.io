# ArcgisEngine实现地图浏览




  
  
#### 全图

```C#
    private void FullExtentTSButton_Click(object sender, EventArgs e)
        {
            mainMapControl.Extent = mainMapControl.FullExtent;
    
        }

```

#### 等比例放大
```C#
        private void btnZoomInStep_Click(object sender, EventArgs e)
        {
            IEnvelope pEnvelope;
            pEnvelope = mainMapControl.Extent;
            pEnvelope.Expand(0.5, 0.5, true);   //放大2倍
            mainMapControl.Extent = pEnvelope;
            mainMapControl.ActiveView.Refresh();

        }
```

#### 等比例缩小
```C#
        private void btnZoomIOutStep_Click(object sender, EventArgs e)
        {
            IActiveView pActiveView = mainMapControl.ActiveView;
            IPoint centerPoint = new PointClass();
            centerPoint.PutCoords((pActiveView.Extent.XMin + pActiveView.Extent.XMax) / 2, (pActiveView.Extent.YMax + pActiveView.Extent.YMin) / 2);
            IEnvelope envlop = pActiveView.Extent;
            envlop.Expand(1.5, 1.5, true);    //与放大的区别在于Expand的参数不同
            pActiveView.Extent.CenterAt(centerPoint);
            pActiveView.Extent = envlop;
            pActiveView.Refresh();

            //如果觉得乱的话，直接修改放大的参数也将就可以 非万不得已不用
            //IEnvelope pEnvelope;
            //pEnvelope = mainMapControl.Extent;
            //pEnvelope.Expand(1.5, 1.5, true);   //放大2倍
            //mainMapControl.Extent = pEnvelope;
            //mainMapControl.ActiveView.Refresh();


        }
```
#### 上一视图
```C#
        // 定义全局变量
        IExtentStack pExtentStack;
        private void PreViewTSButton_Click(object sender, EventArgs e)
        {
            pExtentStack = mainMapControl.ActiveView.ExtentStack;
            //判断是否可以回到前一视图，第一个视图没有前视图
            if (pExtentStack.CanUndo())
            {
                pExtentStack.Undo();      //撤销到上一视图范围
                NextViewTSButton.Enabled = true;   //后一视图可以使用
                if (!pExtentStack.CanUndo())
                {
                    PreViewTSButton.Enabled = false;  //前一视图不能使用
                }
            }
            mainMapControl.ActiveView.Refresh();

        }

```

#### 下一视图
```C#
        private void NextViewTSButton_Click(object sender, EventArgs e)
        {
            pExtentStack = mainMapControl.ActiveView.ExtentStack;
            //判断是否可以回到后一视图，最后一个视图没有后一视图
            if (pExtentStack.CanRedo())  //如果可以重做下一视图
            {
                pExtentStack.Redo();   //重做到下一视图
                PreViewTSButton.Enabled = true;   //上一视图按钮可以使用

                if (!pExtentStack.CanRedo())  //如果不可以重做下一视图
                {
                    NextViewTSButton.Enabled = false;   //下一视图不能用
                }
            }
            mainMapControl.ActiveView.Refresh();
        }
```




---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/arcengine%E5%9C%B0%E5%9B%BE%E6%B5%8F%E8%A7%88%E5%8A%9F%E8%83%BD/  

