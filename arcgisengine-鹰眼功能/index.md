# ArcgisEngine鹰眼功能




  

```c#
        // 鹰眼视图功能  ...
        // OnMapReplaced当mapcontrol1中的地图被替换时，该方法自动加载主空间中所有的图层对象到鹰眼
        private void mainMapControl_OnMapReplaced(object sender, IMapControlEvents2_OnMapReplacedEvent e)
        {
            if (mainMapControl.Map.LayerCount > 0)
            {
                for (int i = 0; i < mainMapControl.Map.LayerCount; i++)
                {
                    // 加载
                    eyeMapControl.AddLayer(mainMapControl.get_Layer(i));
                }
                // 鹰眼视图的范围等于mainMap当前的范围
                eyeMapControl.Extent = mainMapControl.Extent;
                eyeMapControl.ActiveView.Refresh();
            }
        }
```


```c#
        //鹰眼功能   范围更新
        // //--2、在主控件中使用鼠标拖拽视图的时候，鹰眼控件中出现红色矩形框
        //--2、该方法发生在主视图的范围发生变换后，
        private void mainMapControl_OnExtentUpdated(object sender, IMapControlEvents2_OnExtentUpdatedEvent e)
        {

            IEnvelope pEnvelope = (IEnvelope)e.newEnvelope;
            IGraphicsContainer pGraphicsContainer = eyeMapControl.Map as IGraphicsContainer;
            IActiveView pActiveView = pGraphicsContainer as IActiveView;
            pGraphicsContainer.DeleteAllElements();
            IRectangleElement pRectangleEle = new RectangleElementClass();
            IElement pElement = pRectangleEle as IElement;
            pElement.Geometry = pEnvelope;
 
            IRgbColor pColor = new RgbColorClass();
            pColor.Red = 255;
            pColor.Green = 0;
            pColor.Blue = 0;
            pColor.Transparency = 255; 
 
            ILineSymbol pOutline = new SimpleLineSymbolClass();
            pOutline.Width = 3;
            pOutline.Color = pColor;
            pColor = new RgbColorClass();
            pColor.Red = 255;
            pColor.Green = 0;
            pColor.Blue = 0;
            pColor.Transparency = 0;
 
            IFillSymbol pFillSymbol = new SimpleFillSymbolClass();
            pFillSymbol.Color = pColor;
            pFillSymbol.Outline = pOutline;
            IFillShapeElement pFillShapeEle = pElement as IFillShapeElement;
            pFillShapeEle.Symbol = pFillSymbol;
            pGraphicsContainer.AddElement((IElement)pFillShapeEle, 0);
            pActiveView.PartialRefresh(esriViewDrawPhase.esriViewGraphics, null, null);

        }
```

```c#
        // 鹰眼视图功能   鹰眼视图点击=》主图范围跟着变化
        private void eyeMapControl_OnMouseDown(object sender, IMapControlEvents2_OnMouseDownEvent e)
        {
            // 如果被点击
            if (e.button == 1)
            {
                IPoint point = new ESRI.ArcGIS.Geometry.Point();
                //获取点击鹰眼视图的点击的位置
                point.PutCoords(e.mapX, e.mapY);
                // 主图的中心位于点击的点point上
                mainMapControl.CenterAt(point);
                mainMapControl.ActiveView.Refresh();
            }
            // 如果是右键
            else if (e.button == 2)
            {
                IEnvelope pEnv = eyeMapControl.TrackRectangle();
                mainMapControl.Extent = pEnv;
                mainMapControl.ActiveView.Refresh();
            }
        }
```


```c#
        private void eyeMapControl_OnMouseMove(object sender, IMapControlEvents2_OnMouseMoveEvent e)
        {
            if (e.button == 1)
            {
                IPoint pPoint = new PointClass();
                pPoint.PutCoords(e.mapX, e.mapY);
                mainMapControl.CenterAt(pPoint);
                mainMapControl.ActiveView.Refresh();
            }
        }
```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/arcgisengine-%E9%B9%B0%E7%9C%BC%E5%8A%9F%E8%83%BD/  

