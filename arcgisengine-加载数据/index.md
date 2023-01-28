# ArcEngine加载数据





  
### 加载shapefile数据

###### F1 使用AddShapefile方法
```
 private void 添加ShapefileToolStripMenuItem_Click(object sender, EventArgs e)
        {

            try
            {
                //同样实例化一个打开文件的类对象
                OpenFileDialog open = new OpenFileDialog();
                // 如果打开正确
                if (open.ShowDialog() == DialogResult.OK)
                {
                    //首先定义一个空的路径
                    string filePath = string.Empty;
                    // 然后定义一个空的文件名
                    string file = string.Empty;
                    // 获取完整的文件路径
                    string filedir = open.FileName;
                    //如果路径为空，嘛都不返回
                    if (fileDir == "") return;
                    // 对完整路径进行截取  获取最后一个斜杠的索引
                    int pos = filedir.LastIndexOf('\\');
                    // 截取字符串  路径
                    filePath =filedir.Substring(0, pos);
                    //文件名
                    file = filedir.Substring(pos+1);
                    // 需要两个参数
                    axMapControl1.AddShapeFile(filePath, file);
                    //刷新
                    axMapControl1.ActiveView.Refresh();
                }
            }
            catch (Exception)
            {
                MessageBox.Show("请打开正确的文档!", "提醒",MessageBoxButtons.OK, MessageBoxIcon.Error );
            }
        }

```
###### F2  使用工作空间
思路:
1. 创建ShapefileWorkspaceFactory实例，使用OpenFromFile方法打开工作区。
2. 创建FeatureLayer的实例，定义数据集。
3. 使用AddLayer方法加载数据。

```
using ESRI.ArcGIS.Geodatabase;
using ESRI.ArcGIS.DataSourcesFile;
using ESRI.ArcGIS.DataSourcesRaster;
```

```
            OpenFileDialog open = new OpenFileDialog();
            open.CheckFileExists = true;
            open.Title = "打开Shape文件";
            open.RestoreDirectory = true;
            open.Filter = "Shape文件(*.shp)|**.shp";
            open.RestoreDirectory = true;
            open.ShowDialog();

            //获取完整的文件路径 :path+name
            string fileDir = open.FileName;
            //路径为空不返回
            if (fileDir == "") return;
            // 对完整路径进行截取  
            int pos = fileDir.LastIndexOf("\\");
            //  路径
            string pfilePath = fileDir.Substring(0, pos);
            //文件名
            string file = fileDir.Substring(pos + 1);

            //实例化ShapefileWorkspaceFactory工作空间，打开Shape文件
            IWorkspaceFactory pWsFactory = new ShapefileWorkspaceFactoryClass();
            //强制转换   打开路径
            IFeatureWorkspace pWorkSpace = pWsFactory.OpenFromFile(pfilePath, 0) as IFeatureWorkspace;
            //创建并实例化要素类
            IFeatureClass pFeatureClass = pWorkSpace.OpenFeatureClass(file);
            IFeatureLayer pFeatureLayer = new FeatureLayerClass();
            pFeatureLayer.Name = pFeatureClass.AliasName;
            pFeatureLayer.FeatureClass = pFeatureClass;

            mainMapControl.Map.AddLayer(pFeatureLayer);
            mainMapControl.ActiveView.Refresh();
            mainTOCControl.SetBuddyControl(mainMapControl);
```

### 加载栅格数据
主要用到以下2个接口：
1. IRasterPyramid3接口提供对栅格数据集的金字塔属性的访问
- Present属性--判断栅格数据集是否存在金字塔
- Create方法--为栅格数据集创建金字塔
2. IRasterLayer接口继承自ILayer接口
- CreateFromDataset方法--从已有的栅格数据集对象创建图层
- CreateFromRaster方法--从已有的栅格对象创建图层
- Raster属性--获取IRasterLayer接口中的Raster对象
- DisplayResolutionFactor属性--设置栅格数据的分辨率

```
1.直接用IRasterLayer接口打开一个栅格文件并加载到地图控件
IRasterLayer rasterLayer = new RasterLayerClass();
rasterLayer.CreateFromFilePath(fileName); // fileName指存本地的栅格文件路径
axMapControl1.AddLayer(rasterLayer, 0);

```
##### 2.思路:
- (1)使用OpenFromFile方法获得栅格文件的工作区
- (2)使用OpenRasterDataset方法获得栅格文件的数据集
- (3)判断栅格数据集是否具有金字塔
- (4)创建pRasterLayer，并定义数据集
- (5)使用AddLayer方法加载数据
```
            OpenFileDialog pOpenFileDialog = new OpenFileDialog();
            pOpenFileDialog.CheckFileExists = true;
            pOpenFileDialog.Title = "加载栅格数据";
            pOpenFileDialog.Filter = "栅格文件(*.*)|*.bmp;*.tif;*.jpg;*.img;|BMP(*.bmp)|*.bmp;|TIF(*.tif)|*.tif;|JPG(*.jpg)|*.jpg;|IMG(*.img)|*.img";
            pOpenFileDialog.ShowDialog();
            //获取名字
            string pRasterFilename = pOpenFileDialog.FileName;
            if (pRasterFilename == "") return;
            // IO里面获取文件名字与路径的固定用法
            string pPath = System.IO.Path.GetDirectoryName(pRasterFilename);
            string pFileName = System.IO.Path.GetFileName(pRasterFilename);

            IWorkspaceFactory pWorkspaceFactory = new RasterWorkspaceFactory();
            IWorkspace pWorkspace = pWorkspaceFactory.OpenFromFile(pPath, 0);
            IRasterWorkspace pRasterWorkspace = pWorkspace as IRasterWorkspace;
            IRasterDataset pRasterDataset = pRasterWorkspace.OpenRasterDataset(pFileName);
            //影像金字塔的判断与创建
            IRasterPyramid3 pRasPyramid;
            pRasPyramid = pRasterDataset as IRasterPyramid3;
            if (pRasPyramid != null)
            {
                if (!(pRasPyramid.Present))
                { 
                    //创建金字塔
                    pRasPyramid.Create();
                }
            }

            IRaster pRaster;
            pRaster = pRasterDataset.CreateDefaultRaster();
            IRasterLayer pRasterLayer;
            pRasterLayer = new RasterLayerClass();
            pRasterLayer.CreateFromRaster(pRaster);
            ILayer pLayer = pRasterLayer as ILayer;
            mainMapControl.AddLayer(pLayer, 0);

```

### 加载CAD数据
###### 1.作为矢量图层加载  === 1)分图层加载

```
            IWorkspaceFactory pWorkspaceFactory;
            IFeatureWorkspace pFeatureWorkspace;
            pWorkspaceFactory = new CadWorkspaceFactory();
            pFeatureWorkspace = (IFeatureWorkspace)pWorkspaceFactory.OpenFromFile(pPath, 0);
            //加载CAD文件中的线文件
            IFeatureClass pFeatureClass = pFeatureWorkspace.OpenFeatureClass(pFileName + ":polyline");
            IFeatureLayer pFeatureLayer = new FeatureLayerClass();
            pFeatureLayer.Name = pFileName;
            pFeatureLayer.FeatureClass = pFeatureClass;
            mainMapControl.Map.AddLayer(pFeatureLayer);
            mainMapControl.ActiveView.Refresh();
            mainTOCControl.SetBuddyControl(mainMapControl);

```

###### 2.作为矢量图层加载  === 1)整幅图加载
```
            //打开CAD数据集
            IWorkspaceFactory pWorkspaceFactory = new CadWorkspaceFactoryClass();  //using ESRI.ArcGIS.DataSourcesFile;
            IFeatureWorkspace pFeatureWorkspace = pWorkspaceFactory.OpenFromFile(pPath, 0) as IFeatureWorkspace;
            //打开一个要素集
            IFeatureDataset pFeatureDataset = pFeatureWorkspace.OpenFeatureDataset(pFileName);
            // IFeatureContainer可以管理IFeatureDataset中的每个要素类
            IFeatureClassContainer pFeatureContainer = (IFeatureClassContainer)pFeatureDataset;
                
            //对CAD文件中的要素进行遍历处理
            for (int i = 0; i < pFeatureContainer.ClassCount; i++)
            {
                IFeatureClass pFeatureClass = pFeatureContainer.get_Class(i);
                //如果是注记， 则添加注记层
                if (pFeatureClass.FeatureType == esriFeatureType.esriFTCoverageAnnotation)
                {
                    IFeatureLayer pFeatureLayer = new FeatureLayerClass();
                    pFeatureLayer.Name = pFeatureClass.AliasName;
                    pFeatureLayer.FeatureClass = pFeatureClass;
                    mainMapControl.Map.AddLayer(pFeatureLayer);       
                }
                else  //如果是点线面则添加要素层
                {
                    IFeatureLayer pFeatureLayer = new FeatureLayerClass();
                    pFeatureLayer.Name = pFeatureClass.AliasName;
                    pFeatureLayer.FeatureClass = pFeatureClass;
                    mainMapControl.Map.AddLayer(pFeatureLayer);

                 }
                mainMapControl.ActiveView.Refresh();
                mainTOCControl.SetBuddyControl(mainMapControl);
            }

```

###### F3.作为栅格图层加载 
```
            IWorkspaceFactory pWorkspaceFactory;
            IWorkspace pWorkspace;
            ICadDrawingDataset pCadDrawingDataset;
            ICadDrawingWorkspace pCadDrawingWorkspace;
            ICadLayer pCadLayer;

            pWorkspaceFactory = new CadWorkspaceFactoryClass();
            pWorkspace = pWorkspaceFactory.OpenFromFile(pPath, 0);
            pCadDrawingWorkspace = (ICadDrawingWorkspace)pWorkspace;
            //获得CAD文件的数据集
            pCadDrawingDataset = pCadDrawingWorkspace.OpenCadDrawingDataset(pFileName);
            pCadLayer = new CadLayerClass();
            pCadLayer.CadDrawingDataset = pCadDrawingDataset;
            mainMapControl.Map.AddLayer(pCadLayer);
            mainMapControl.ActiveView.Refresh();
            mainTOCControl.SetBuddyControl(mainMapControl);

```

###### 加载个人地理数据库
```


```
###### 加载文件地理数据库
```


```















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/arcgisengine-%E5%8A%A0%E8%BD%BD%E6%95%B0%E6%8D%AE/  

