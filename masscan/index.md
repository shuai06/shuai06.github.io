# masscan




> masscan号称是世界上最快的扫描软件，可以在3分钟内扫描整个互联网端口，但是这个是由条件的4核电脑，双端口10G网卡



项目地址： https://github.com/robertdavidgraham/masscan



#### 安装

###### Linux上

```bash
sudo apt-get install git gcc make libpcap-dev
git clone https://github.com/robertdavidgraham/masscan
cd masscan
make
```



###### Windows上

因为我用的Linux比较多，所用没安过Windows的，请参考：https://www.cnblogs.com/flaray/p/11213730.html



#### 使用

###### 常用命令

```bash
masscan 172.16.0.0/16 -p0-65535 --banners --append-output  -oL scan_result.list  --max-rate 10000


# 扫描ip列表文件
masscan -iL ip.txt -p0-65535 --banners --append-output  -oL scan_result.list  --max-rate 10000


# 输出json格式     -Oj
masscan -iL ip.txt -p80,443,3306 --banners --append-output  -oJ result.json  --max-rate 10000

# Note: 发包速率(--max-rate)不要太大，且Linux下比Windows速度快



--adapter-ip 	# 指定发包的IP地址
--adapter-port 	# 指定发包的源端口
--adapter-mac	# 指定发包的源MAC地址
--router-mac 	# 指定网关的MAC地址
--exclude 		# IP地址范围黑名单,防止masscan扫描
--excludefile	# 指定IP地址范围黑名单文件
--includefile,-iL # 读取一个范围列表进行扫描
--wait			# 指定发送完包之后的等待时间,默认为10秒
```





###### Python脚本解析masscan扫描结果

> 是基于json输出格式(-OJ)的

```python
# Author：https://www.jianshu.com/p/b6edaa3acbbf

import json
from openpyxl import Workbook
import xlsxwriter
import socket
def get_list(filepath):
    f = open(filepath,encoding='utf-8')
    c = json.load(f)
    list = []
    for i in c:
        ip = i['ip']
        port = str(i['ports'][0]['port'])
        status = 'open'
        try:
            if i['ports'][0]['service']:
                name = i['ports'][0]['service']['name']
                banner = str(i['ports'][0]['service']['banner'])
        except:
            name = ''
            banner = ''
        line = [ip,port,status,name,banner]
        list.append(line)
    return list
def quchong(l1):
    l2 =[]
    for data1 in l1:
        for data2 in l1:
            if data1[0]==data2[0] and data1[1]==data2[1]:
                if data1[3] ==''and data2[3] !='':
                    # print(data1,data2)
                    l2.append(data1)
    for i in l2:
        try:
            l1.remove(i)
        except:pass
    l1 = [list(t) for t in set(tuple(_) for _ in l1)]
    return l1

def write_excle(list):
    f = xlsxwriter.Workbook('port.xlsx')
    worksheet1 = f.add_worksheet('扫描信息')
    worksheet2 = f.add_worksheet('主机ip列表')
    worksheet1.write(0, 0, 'ip')
    worksheet1.write(0, 1, '端口')
    worksheet1.write(0, 2, '状态')
    worksheet1.write(0, 3, '服务')
    worksheet1.write(0, 4, 'banner')
    worksheet2.write(0, 0, '主机ip')
    newlist= []
    for i in list:
        newlist.append(i[0])
    newlist=set(newlist)
    total1 = 0
    total2 = len(newlist)
    newlist=sorted(newlist, key=socket.inet_aton)
    for index, p in enumerate(list):
        total1+=1
        for j, q in enumerate(p):
            worksheet1.write(index + 1, j, q)
    for index, p in enumerate(newlist):
        worksheet2.write(index + 1, 0, p)
    f.close()
    return total1,total2
if __name__ == '__main__':
    filepath = 'C:/1/result.json'  #填写要解析masscan扫描json格式报告的文件路径
    result = get_list(filepath)
    result = quchong(result)
    sum = write_excle(result)
    print('共检测到存活主机%d个，端口信息%d条'% (sum[1],sum[0]))

```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/masscan/  

