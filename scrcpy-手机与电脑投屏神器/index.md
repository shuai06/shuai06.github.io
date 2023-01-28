# scrcpy-手机与电脑投屏神器

  
  
  
#### scrcpy 安装

```
snap install scrcpy
```

#### adb服务安装
  
```
sudo apt-get install android-tools-adb
```

#### adb配置
  
###### 手机通过USB连接电脑

```
lsusb
```

找到自己手机的识别号,(可以对比数据线插入之前和插入之后多了哪个就是哪个)

###### 创建设备文件

下面所有的`04e8`改成自己的识别号, `android.rules`文件名可自定义

```bash
mkdir ~/.android
# 注意你的设备号
echo 0x04e8 > ~/.android/adb_usb.ini
sudo touch /etc/udev/rules.d/android.rules
sudo gedit /etc/udev/rules.d/android.rules

```

###### 在文件中输入:

注意自己的设备号

```
SUBSYSTEM=="usb", SYSFS{idVendor}=="你的设备号", MODE="0666"
```

###### 保存后修改文件权限

```bash
sudo chmod 777 /etc/udev/rules.d/android.rules
```

###### 启动adb服务

```bash
service udev restart
adb start-server
adb devices
```

有设备就说明成功了, 如果没有看看自己手机的开发者模式以及USB调试有没有打开



#### **使用scrcpy**

命令行输入

```python
# 直接使用
scrcpy
# 设置显示尺寸为1080
nohun scrcpy -m 1080 &  
```

就会弹出界面了



###### scrcpy使用方法

```python
"""
鼠标左键点击、滑动;
长按鼠标中键回到主屏幕;Ctrl+Shift+V
鼠标右键返回复制文本电脑到手机:  电脑上复制后, 在手机投屏界面按Ctrl+Shift+V复制到手机剪切板, 然后手机中粘贴手机到电脑: 手机上复制到剪切板中, 在投屏界面按下Ctrl+C键，再到电脑正常上粘贴传输文件: 直接在文件管理器复制粘贴

"""
```



#### 快捷键

###### 全屏/回到合适尺寸

`ctrl +f`    `ctrl + x`



###### 展开/折叠通知栏

`ctrl +n`   `ctrl + shift + n`



#### 结束投屏

`scrcpy -S`

或者关掉显示窗口即可

---

#### Scrcpy 的命令参数

|                             |                                                              |
| --------------------------- | ------------------------------------------------------------ |
| **关闭手机屏幕**            | `scrcpy -S`                                                  |
| **限制画面分辨率**          | `scrcpy -m 1024` (比如限制为 1024)                           |
| **修改视频码率**            | `scrcpy -b 4M` (默认 8Mbps，改成 4Mbps)                      |
| **裁剪画面**                | `scrcpy -c 1224:1440:0:0` 表示分辨率 1224x1440 并且偏移坐标为 (0,0) |
| **多设备切换**              | `scrcpy -s 设备ID` (使用 `adb devices` 命令查看设备ID)       |
| **窗口置顶**                | `scrcpy -T`                                                  |
| **显示触摸点击**            | `scrcpy -t` 在演示或录制教程时，可在画面上对应显示出点击动作 |
| **全屏显示**                | `scrcpy -f`                                                  |
| **文件传输默认路径**        | `scrcpy --push-target /你的/目录` 将文件拖放到 scrcpy 可以传输文件，此命令指定默认保存目录 |
| **只读模式(仅显示不控制)**  | `scrcpy -n`                                                  |
| **屏幕录像**                | `scrcpy -r 视频文件名.mp4` 或 `.mkv`                         |
| **屏幕录像 (禁用电脑显示)** | `scrcpy -Nr 文件名.mkv`                                      |
| **设置窗口标题**            | `scrcpy --window-title '异次元好棒！'`                       |
| **同步传输声音**            | 可借助 [USBaudio](https://github.com/rom1v/usbaudio) 这个开源项目实现，但仅支持 [Linux](https://www.iplaysoft.com/os/linux-platform) 系统 |



#### Scrcpy 快捷键列表

|                                |                            |
| ------------------------------ | -------------------------- |
| 切换全屏模式                   | `Ctrl`+`F`                 |
| 将窗口调整为1：1（完美像素）   | `Ctrl`+`G`                 |
| 调整窗口大小以删除黑色边框     | `Ctrl`+`X` \| 双击黑色背景 |
| 设备 `HOME` 键                 | `Ctrl`+`H` \| 鼠标中键     |
| 设备 `BACK` 键                 | `Ctrl`+`B` \| 鼠标右键     |
| 设备 `任务管理` 键 (切换APP)   | `Ctrl`+`S`                 |
| 设备 `菜单` 键                 | `Ctrl`+`M`                 |
| 设备`音量+`键                  | `Ctrl`+`↑`                 |
| 设备`音量-`键                  | `Ctrl`+`↓`                 |
| 设备`电源键`                   | `Ctrl`+`P`                 |
| 点亮手机屏幕                   | 鼠标右键                   |
| 复制内容到设备                 | `Ctrl`+`V`                 |
| 启用/禁用 FPS 计数器（stdout） | `Ctrl`+`i`                 |
| 安装APK                        | 将 apk 文件拖入投屏        |
| 传输文件到设备                 | 将文件拖入投屏（非apk）    |



- **投屏并录屏：**`scrcpy -r file.mp4`
- **不投屏只录屏：**`scrcpy -Nr file.mp4`



###### 录制声音

结合：https://github.com/rom1v/usbaudio



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/scrcpy-%E6%89%8B%E6%9C%BA%E4%B8%8E%E7%94%B5%E8%84%91%E6%8A%95%E5%B1%8F%E7%A5%9E%E5%99%A8/  

