# ubuntu server-Rkhunter




## 谣言

- Linux系统很安全，从不感染病毒
- 已知的恶意软件没有Windows系统多



## Rootkit

- 隐藏自身能力超强的恶意软件(一 个或一组)
- 将自身注入主板芯片、硬盘引导扇区、系统内核、底层驱动
- 运行级别优先于系统或驻留于内核特权层级
- 难于发现、难于清除、更换硬件都无法清除
- 最早是出现在Unix系统上的一类正向技术
- 有时木马程序也可视为Rootkit 的一种





## Rootkit Hunter

- 安全监视和分析工具(非处置型工具)
- 检测Rootkit、恶意软件、不安全设置、文件完整性、可疑端口
- 基于特征匹配
- 脚本(调用其他通用工具完成检测任务)
- 运行方式
- 需要要root权限
- 只能运行于Bourne类型的shell (bash、 ksh)
- 手动运行、cron job



**安装**

```bash
sudo apt install rkhunter
# 按照向导即可
```

**配置**

```bash
sudo vi /etc/default/rkhunter
    CRON DAILY_RUN="true"	# 每天自动运行来检查系统
    CRON_ DB_ UPDATE="true"
# 这里简单修改几项
```

**升级特征库**

```bash
sudo rkhunter --propupd
```

**执行检查**

```bash
# 默认对系统所有检查项进行（每一项都需要回车确认，--sk就不用手动敲回车了）
sudo rkhunter --check --sk
# 检查结果如果显示是绿色的，基本就没啥问题
```

**日志文件**

```bash
# var/log/rkhunter.log
less var/log/rkhunter.log
```

**误报白名单**

```bash
sudo vi /etc/rkhunter.conf
    SCRIPTWHITELIST=/usr/bin/egrep
    SCRIPTWHITELIST=/usr/bin/fgrep
    SCRIPTWHITELIST=/usr/bin/ldd
```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/ubuntu-server-rkhunter/  

