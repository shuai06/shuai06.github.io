# ArcEngine初识 -- 打开地图



  
### 一句代码

1. 创建winform程序
1. 必须要添加```LicenseControl```控件
1.  ```TOCControl```视图控件  => 属性,设置关联属性绑定(Buddy)到地图控件上
1.  ```ToolbalControl```工具条控件   伙伴控价  => 设置关联属性绑定到地图控件上
1.  ```MapControl```地图控件  也可以设置属性设置默认打开的地图
1. 引用```ESRI.ArcGIS.Version```
1. 在主程序代码中加入：
```
//在窗口加载前添加    绑定产品码
// 注意是 program.cs文件
 ESRI.ArcGIS.RuntimeManager.Bind(ESRI.ArcGIS.ProductCode.EngineOrDesktop);
```
才能正常运行

### 常见的AE控价
- PageLayoutControl 布局视图
- TOOControl 内容列表
- MapControl 数据视图
- 

### 打开地图文档
###### F1 自己写的
```         c#
//打开mxd文档 F1
            try
            {
                //实例化一个对象
                OpenFileDialog open = new OpenFileDialog();
                //设置参数
                //检查文件是否存在
                open.CheckFileExists = true;
                open.Title = "打开地图文档";
                // 不允许多个文件同时打开
                open.Multiselect = false;
                // 存储时打开的文件路径
                open.RestoreDirectory = true;
                //打开文件格式
                open.Filter = "地图文档(*.mxd)|*.mxd;|ArcMap模板(*.mxt)|*.mxt";|所有地图格式(*.mxd;*.mxt;*.pmf)|*.mxd;*.mxt;*.pmf"/文件过滤条件
                open.Multiselect = false;//禁止多选
                open.RestoreDirectory = true;  // 存储打开的文件路径
                //判断如果文件正常打开
                if (open.ShowDialog() == DialogResult.OK)
                {
                    // 获取文件路径  path+name
                    string fileName = open.FileName;
                    // 数据窗口加载地图文档
                    axMapControl1.LoadMxFile(fileName);
                    // 刷新一下页面
                    axMapControl1.ActiveView.Refresh();
                    // 伙伴控件可以通过下面的方式来进行绑定
                    axTOCControl1.SetBuddyControl(axMapControl1);

                }
            }
            catch(Exception)
            {
                MessageBox.Show("请打开正确的文档!", "提醒", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }

        }
       
```
###### F2 使用IMapControl接口的LoadMxFile方法
```c#
            OpenFileDialog open = new OpenFileDialog();
            //设置参数
            //检查文件是否存在
            open.CheckFileExists = true;
            open.Title = "打开地图文档";
            // 不允许多个文件同时打开
            open.Multiselect = false;
            // 存储时打开的文件路径
            open.RestoreDirectory = true;
            //打开文件格式
            open.Filter = "地图文档(*.mxd)|*.mxd;|ArcMap模板(*.mxt)|*.mxt;|所有地图格式(*.mxd;*.mxt;*.pmf)|*.mxd;*.mxt;*.pmf";//文件过滤条件
            open.Multiselect = false;//禁止多选
            open.RestoreDirectory = true;  // 存储打开的文件路径
            open.ShowDialog();
            //文件名
            string fileName = open.FileName;

            if (fileName == "")
            {
                return;
            }
            //检查地图文档有效性
            if (mainMapControl.CheckMxFile(fileName))
            {
                mainMapControl.LoadMxFile(fileName);
                mainMapControl.ActiveView.Refresh();

            }
            else
            {
                MessageBox.Show(fileName + "是无效的地图文档!", "信息提示");
                return;
            }

```
###### F3 使用IMapDocument接口

```c#
// using ESRI.ArcGIS.Carto;
            OpenFileDialog open = new OpenFileDialog();
            //设置参数
            //检查文件是否存在
            open.CheckFileExists = true;
            open.Title = "打开地图文档";
            // 不允许多个文件同时打开
            open.Multiselect = false;
            // 存储时打开的文件路径
            open.RestoreDirectory = true;
            //打开文件格式
            open.Filter = @"地图文档(*.mxd)|*.mxd;|ArcMap模板(*.mxt)|*.mxt;|所有地图格式(*.mxd;*.mxt;*.pmf)|*.mxd;*.mxt;*.pmf";//文件过滤条件
            open.Multiselect = false;//禁止多选
            open.RestoreDirectory = true;
            //打开窗口
            open.ShowDialog();
            // 存储打开的文件路径
            string fileName = open.FileName;
            if (mainMapControl.CheckMxFile(fileName))
            {
                //将数据载入pMapDocument并与Map控件关联
                IMapDocument pMapDocument = new MapDocument();   // using ESRI.ArcGIS.Carto;
                pMapDocument.Open(fileName, "");
                //获取Map地图中激活的地图文档
                mainMapControl.Map = pMapDocument.ActiveView.FocusMap;
                mainMapControl.ActiveView.Refresh();
            }

```
###### F4 使用ControlsOpenDocCommandClass类库资源 =》 适合比赛用
```c#
//using ESRI.ArcGIS.Controls;
            ICommand command = new ControlsOpenDocCommandClass();    //using ESRI.ArcGIS.Controls;
            command.OnCreate(mainMapControl.Object);
            command.OnClick();
            mainTOCControl.SetBuddyControl(mainMapControl);

```

##### 待改进之处： 
1. 窗口的大小以及出现的位置，美化
2. Form的name随着文档的名字改变

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/arcgisengine-%E6%89%93%E5%BC%80%E6%96%87%E6%A1%A3/  

