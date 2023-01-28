# nbtscan




> 当我们处于局域网中

> 用来探测内网存活主机



kali中已经安装了：

```bash
root@xps:~# whereis nbtscan
nbtscan: /usr/bin/nbtscan /usr/share/man/man1/nbtscan.1.gz
root@xps:~# nbtscan 

NBTscan version 1.6.
This is a free software and it comes with absolutely no warranty.
You can use, distribute and modify it under terms of GNU GPL 2+.

Usage:
nbtscan [-v] [-d] [-e] [-l] [-t timeout] [-b bandwidth] [-r] [-q] [-s separator] [-m retransmits] (-f filename)|(<scan_range>) 
	-v		verbose output. Print all names received
			from each host
	-d		dump packets. Print whole packet contents.
	-e		Format output in /etc/hosts format.
	-l		Format output in lmhosts format.
			Cannot be used with -v, -s or -h options.
	-t timeout	wait timeout milliseconds for response.
			Default 1000.
	-b bandwidth	Output throttling. Slow down output
			so that it uses no more that bandwidth bps.
			Useful on slow links, so that ougoing queries
			don't get dropped.
	-r		use local port 137 for scans. Win95 boxes
			respond to this only.
			You need to be root to use this option on Unix.
	-q		Suppress banners and error messages,
	-s separator	Script-friendly output. Don't print
			column and record headers, separate fields with separator.
	-h		Print human-readable names for services.
			Can only be used with -v option.
	-m retransmits	Number of retransmits. Default 0.
	-f filename	Take IP addresses to scan from file filename.
			-f - makes nbtscan take IP addresses from stdin.
	<scan_range>	what to scan. Can either be single IP
			like 192.168.1.1 or
			range of addresses in one of two forms: 
			xxx.xxx.xxx.xxx/xx or xxx.xxx.xxx.xxx-xxx.
Examples:
	nbtscan -r 192.168.1.0/24
		Scans the whole C-class network.
	nbtscan 192.168.1.25-137
		Scans a range from 192.168.1.25 to 192.168.1.137
	nbtscan -v -s : 192.168.1.0/24
		Scans C-class network. Prints results in script-friendly
		format using colon as field separator.
		Produces output like that:
		192.168.0.1:NT_SERVER:00U
		192.168.0.1:MY_DOMAIN:00G
		192.168.0.1:ADMINISTRATOR:03U
		192.168.0.2:OTHER_BOX:00U
		...
	nbtscan -f iplist
		Scans IP addresses specified in file iplist.
		

# 参数
-a	列出为其主机名提供的远程计算机名字表。
-A	列出为其IP地址提供的远程计算机名字表。
-c	列出包括了IP地址的远程名字高速缓存器。
-n	列出本地NetBIOS名字。
-r	列出通过广播和WINS解析的名字。
-R	消除和重新加载远程高速缓存器名字表。
-S	列出有目的地IP地址的会话表。
-s	列出会话表对话。
```



#### 示例

```bash
## 以：分割显示

## Linux上

nbtscan -v -s : 192.168.117.130

# 扫描整个C段
nbtscan -r 192.168.1.0/24

# 扫描一个范围
nbtscan 192.168.1.25-137


# 从文件读取扫描范围
nbtscan -f <File>


## Windows上
nbtscan.exe -m 192.168.1.0/24
```



#### 高级用法

```bash
nbtscan -v -s ' ' 192.168.117.130
 
nbtscan -v -s ' ' 192.168.117.130 | awk '{print $1}' |uniq
```







#### 这个工具的运行流程：

遍历输入的IP范围，以广播MAC地址发送ARP查询，一旦接收到ARP回复，遍记录相应的IP与MAC地址，同时向对方发送NBNS消息查询对方的主机信息打印出每条信息





其他探测内网主机的方法：http://www.360doc.com/content/19/1129/18/13328254_876375955.shtml

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/nbtscan/  

