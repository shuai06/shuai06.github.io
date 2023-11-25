# ArcEngine文件保存功能








### 地图文档保存

###### 问题：空文档也能保存
```c#
private void saveToolStrip_Click(object sender, EventArgs e)
        {
            try
            {
                string sMxdFileName = mainMapControl.DocumentFilename;
                IMapDocument pMapDocument = new MapDocumentClass();
                //检查文档是否为空以及有效性
                if (sMxdFileName != null && mainMapControl.CheckMxFile(sMxdFileName))
                {
                    if (pMapDocument.get_IsReadOnly(sMxdFileName))
                    {
                        MessageBox.Show("地图本当为只读，不能保存！");
                        pMapDocument.Close();
                        return;
                    }     
                    else
                    {
                    SaveFileDialog pSaveFileDialog = new System.Windows.Forms.SaveFileDialog();
                    pSaveFileDialog.Title = "请选择保存路径";
                    pSaveFileDialog.Filter = "ArcMap文档(*.mxd)|*.mxd|ArcMap模板(*.mxt)|*.mxt";
                    //当相同的文件存在是提示错误
                    pSaveFileDialog.OverwritePrompt = true;
                    pSaveFileDialog.RestoreDirectory = true;
                    if (pSaveFileDialog.ShowDialog() == DialogResult.OK)
                    {
                        //获取名字
                        sMxdFileName = pSaveFileDialog.FileName;
                    }
                    else
                    {
                        return;
                    }
                        pMapDocument.New(sMxdFileName);
                        pMapDocument.ReplaceContents(mainMapControl.Map as IMxdContents);
                        //保存为绝对路径
                        pMapDocument.Save(pMapDocument.UsesRelativePaths, true);
                        pMapDocument.Close();
                        MessageBox.Show("保存文档成功");
                }
                }
               
                
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }

```
### 地图文档另存为1

测试

```c#
private void saveAsToolStrip_Click(object sender, EventArgs e)
        {
            try
            {
                SaveFileDialog pSaveDialog = new System.Windows.Forms.SaveFileDialog();
                pSaveDialog.Title = "另存为";
                pSaveDialog.OverwritePrompt = true;//当相同的文件存在是提示错误
                pSaveDialog.Filter = "ArcMap文档(*.mxd)|*.mxd|ArcMap模板(*.mxt)|*.mxt";
                pSaveDialog.RestoreDirectory = true;
                if (pSaveDialog.ShowDialog() == DialogResult.OK)
                {
                    string sFilePath = pSaveDialog.FileName;
                    IMapDocument pMapDocument = new MapDocumentClass();
                    pMapDocument.New(sFilePath);
                    pMapDocument.ReplaceContents(mainMapControl.Map as IMxdContents);
                    pMapDocument.Save(true, true);
                    pMapDocument.Close();
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
```

### 地图另存为2
```c#
                ICommand command = new ControlsSaveAsDocCommandClass();
                command.OnCreate(mainMapControl.Object);
                command.OnClick();
```

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/arcgisengine-%E6%96%87%E4%BB%B6%E7%9A%84%E4%BF%9D%E5%AD%98%E4%B8%8E%E5%8F%A6%E5%AD%98/  

