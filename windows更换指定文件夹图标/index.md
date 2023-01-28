# Windows更换指定文件夹图标


### 前景提示

小学弟要我写这种奇奇怪怪的东西，奇奇怪怪的东西.... :laughing:



![哈哈哈](http://image.xpshuai.cn/hahah.png)

好了，那就写写吧，谁让我这么可爱呢（小菜鸡瑟瑟发抖） :sweat_smile:





### 正题

先画个界面（哈哈指向画界面不行写代码）

![](http://image.xpshuai.cn/formui.png)



贴上垃圾的code：

```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Diagnostics;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace 更换指定文件夹图标
{
    public partial class Form1 : Form
    {

        public string foldPath;
        public string icoPath;
        public string desktopiniPath;

        public Form1()
        {
            InitializeComponent();
        }

        // 选择要更换的文件夹
        private void textBox1_MouseClick(object sender, MouseEventArgs e)
        {
            try
            {
                this.textBox1.Text = "";
                FolderBrowserDialog dialog = new FolderBrowserDialog();
                dialog.Description = "请选择文件路径";
                if (dialog.ShowDialog() == DialogResult.OK)
                {
                    foldPath = dialog.SelectedPath;
                }

                //System.Diagnostics.Process.Start("Explorer.exe", "c:\\windows");
                this.textBox1.Text = foldPath;
            
            }
            catch (Exception)
            {
                MessageBox.Show("请正确选择图标文件!", "提醒", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }

        }

        //选择要更换的图标
        private void textBox2_MouseClick(object sender, MouseEventArgs e)
        {
            try
            {
                this.textBox2.Text = "";
                OpenFileDialog open = new OpenFileDialog();
                open.CheckFileExists = true;
                open.Multiselect = false;
                open.RestoreDirectory = true;
                open.Multiselect = false;
                open.RestoreDirectory = true; 
                open.Filter = "图标文件(*.*)|*.ico";

                if (open.ShowDialog() == System.Windows.Forms.DialogResult.OK)
                {
                    // 获取文件路径 
                    icoPath = open.FileName;
                    this.textBox2.Text = icoPath;

                }
            }
            catch (Exception)
            {
                MessageBox.Show("请正确选择图标文件!", "提醒", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }

        private void button1_Click(object sender, EventArgs e)
        {
            if (textBox1.Text != "" && textBox2.Text != "" && textBox3.Text != "" && textBox4.Text != "")
            {
                //1.新建desktop.ini文件
                desktopiniPath = foldPath + "\\desktop.ini";          //设置文件路径
                if (!File.Exists(desktopiniPath))
                {
                    FileStream fs1 = new FileStream(desktopiniPath, FileMode.Create, FileAccess.Write);//创建写入文件
                    StreamWriter sw = new StreamWriter(fs1);

                    sw.WriteLine("[.ShellClassInfo]");//开始写入值
                    sw.WriteLine("InfoTip=" + textBox3.Text);
                    sw.WriteLine("LocalizedResourceName=" + textBox4.Text);
                    sw.WriteLine("");
                    sw.WriteLine("IconFile=" + icoPath);
                    sw.WriteLine("IconIndex=mainicon");
                    sw.WriteLine("");

                    sw.WriteLine("[[ExtShellFolderViews]]");
                    sw.WriteLine("[{BE098140-A513-11D0-A3A4-00C04FD706EC}]");
                    sw.WriteLine("IconArea_Text=0x000000FF");
                    sw.WriteLine("");
                    sw.WriteLine("[DeleteOnCopy]");
                    sw.WriteLine("Owner=Temp");
                    sw.WriteLine("PersonalizedName=My test file");
                    sw.WriteLine("");

                    sw.WriteLine("[ViewState]");
                    sw.WriteLine("Mode=");
                    sw.WriteLine("Vid=");
                    sw.WriteLine("FolderType=Generic");
                    sw.Close();
                    fs1.Close();

                }
            }
            else
            {
                MessageBox.Show("请输入完整文本框内容!", "提醒", MessageBoxButtons.OK, MessageBoxIcon.Information);
            
            }


            //2.
            Process CmdProcess = new Process();
            CmdProcess.StartInfo.FileName = "cmd.exe";
            CmdProcess.StartInfo.WorkingDirectory = foldPath;  // cmd打开的目录
            CmdProcess.StartInfo.CreateNoWindow = true;         // 不创建新窗口    
            CmdProcess.StartInfo.UseShellExecute = false;       //不启用shell启动进程  
            CmdProcess.StartInfo.RedirectStandardInput = true;  // 重定向输入    
            CmdProcess.StartInfo.RedirectStandardOutput = true; // 重定向标准输出    
            CmdProcess.StartInfo.RedirectStandardError = true;  // 重定向错误输出  
            //2.1隐藏desktop.ini文件
             // attrib +h temp/desktop.ini
            // attrib +s temp
            DirectoryInfo pathInfo = new DirectoryInfo(foldPath);
            //string newPath = pathInfo.Parent.FullName;  //父目录名
            int index = foldPath.LastIndexOf('\\');
            //从下一个索引开始截取
            string desktopName = foldPath.Substring(index + 1);  //后面文件名

            CmdProcess.Start();//执行  
            CmdProcess.StandardInput.WriteLine("attrib +h " + desktopiniPath + "&& cd .. && attrib +s " + desktopName + " && exit");
            CmdProcess.StandardInput.AutoFlush = true;
            CmdProcess.WaitForExit();//等待程序执行完退出进程  
            CmdProcess.Close();//结束 

            MessageBox.Show("OK!", "提示", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }
    }
}

```



**链接：**

https://github.com/shuai06/ChangeDirLogo



### BUG

完成之后确实更改了，但是有一定延迟......:cry:



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/windows%E6%9B%B4%E6%8D%A2%E6%8C%87%E5%AE%9A%E6%96%87%E4%BB%B6%E5%A4%B9%E5%9B%BE%E6%A0%87/  

