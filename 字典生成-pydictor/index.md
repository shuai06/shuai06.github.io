# 字典生成-pydictor


>  一个强大实用的黑客暴力破解字典建立工具



地址：https://github.com/LandGrey/pydictor.git



#### 简单示例

生成字典

```bash
python pydictor.py --sedb
```

合并去重

```bash
python pydictor.py -tool uniqbiner /my/all/dict/
```



多字典文件组合工具

```bash
python pydictor.py -tool hybrider heads.txt some_others.txt tails.txt
```



>*使用pydictor有个窍门： 时刻清楚你想要什么样子的字典.*

#### 

#### pydictor可以生成的所有字典的类型及其说明

| 归属   | 类别         | 标识符 | 描述                      | 支持功能代号 |
| ------ | ------------ | ------ | ------------------------- | ------------ |
| core   | base         | C1     | 基础字典                  | F1 F2 F3 F4  |
| core   | char         | C2     | 自定义字符集字典          | F1 F2 F3 F4  |
| core   | chunk        | C3     | 排列组合字典              | ALL          |
| core   | conf         | C4     | 配置语法生成字典          | ALL          |
| core   | **extend**   | C5     | 规则扩展字典              | ALL          |
| core   | sedb         | C6     | 社会工程学字典            | ALL          |
| tool   | combiner     | T1     | 字典合并工具              |              |
| tool   | comparer     | T2     | 字典比较相减工具          | ALL          |
| tool   | **counter**  | T3     | 词频统计工具              | ALL          |
| tool   | **handler**  | T4     | 筛选处理原有字典工具      | ALL          |
| tool   | uniqbiner    | T5     | 先合并后去重工具          | ALL          |
| tool   | **uniqifer** | T6     | 字典去重工具              | ALL          |
| tool   | **hybrider** | T7     | 多字典文件组合工具        | F1 F2 F3 F4  |
| plugin | birthday     | P1     | 生日日期字典插件          | ALL          |
| plugin | ftp          | P2     | 关键词生成ftp密码字典插件 | ALL          |
| plugin | pid4         | P3     | 身份证后四位字典插件      | ALL          |
| plugin | pid6         | P4     | 身份证后六位字典插件      | ALL          |
| plugin | pid8         | P5     | 身份证后八位字典插件      | ALL          |
| plugin | scratch      | P6     | 网页原始关键词字典插件    | ALL          |



#### 字典操作功能及说明对照表

| 功能   | 功能代号 | 说明                                     |
| ------ | -------- | ---------------------------------------- |
| len    | F1       | 定义长度范围                             |
| head   | F2       | 添加前缀                                 |
| tail   | F3       | 添加后缀                                 |
| encode | F4       | 编码或自定义加密方法                     |
| occur  | F5       | 字母、数字、特殊字符出现次数范围筛选     |
| types  | F6       | 字母、数字、特殊字符各种类数范围筛选     |
| regex  | F7       | 正则筛选                                 |
| level  | F8       | 字典级别筛选                             |
| leet   | F9       | 1337 模式                                |
| repeat | F10      | 字母、数字、特殊字符连续出现次数范围筛选 |



#### 支持的编码或加密方式

| 方式   | 描述                                       |
| ------ | ------------------------------------------ |
| none   | 默认方式, 不进行任何编码                   |
| b16    | base16 编码                                |
| b32    | base32 编码                                |
| b64    | base64 编码                                |
| des    | des 算法, 需要根据情况修改代码             |
| execjs | 执行本地或远程js函数, 需要根据情况修改代码 |
| hmac   | hmac 算法, 需要根据情况修改代码            |
| md5    | md5 算法输出32位                           |
| md516  | md5 算法输出16位                           |
| rsa    | rsa 算法 需要根据情况修改代码              |
| sha1   | sha-1 算法                                 |
| sha256 | sha-256 算法                               |
| sha512 | sha-512 算法                               |
| url    | url 编码                                   |
| test   | 一个自定义编码方法的示例                   |

```bash
 --encode b64
```



#### occur 功能

```
用法 : --occur [字母出现次数的范围] [数字出现次数的范围] [特殊字符出现次数的范围]
示例: --occur ">=4" "<6" "==0"
```

#### types 功能

```
用法 : --types [字母种类的范围] [数字种类的范围] [特殊字符种类的范围]
示例: --types "<=8" "<=4" "=0"
```

#### repeat 功能

```
用法 : --repeat [字母连续出现次数范围] [数字连续出现次数范围] [特殊字符连续出现次数范围]
示例: --repeat "<=3" ">=3" "==0"
```

#### regex 功能

```
用法 : --regex [正则表达式]
示例: --regex "^z.*?g$"
```

#### level 功能

```
用法 : --level [level]
示例: --level 4      /funcfg/extend.conf配置文件中level大于等于4的项目会被启用
```

#### leet功能

#### 默认置换表

```
leet字符 = 替换字符，可以修改/funcfg/leet_mode.conf更改替换表
a = 4
b = 6
e = 3
l = 1
i = 1
o = 0
s = 5
```

#### 模式代码

```
0               默认模式，全部替换
1               从左至右, 将第一个遇到的leet字符全部替换
2               从右至左, 将第一个遇到的leet字符全部替换
11-19           从左至右, 将第一个遇到的leet字符最多替换 code-10 个
21-29           从右至左, 将第一个遇到的leet字符最多替换 code-20 个
```

###### 代码作用表

| 代码 | 原字符串        | 被替换后的新字符串 |
| ---- | --------------- | ------------------ |
| 0    | as a airs trees | 45 4 41r5 tr335    |
| 1    | as a airs trees | 4s 4 4irs trees    |
| 2    | as a airs trees | a5 a air5 tree5    |
| 11   | as a airs trees | 4s a airs trees    |
| 12   | as a airs trees | 4s 4 airs trees    |
| 13   | as a airs trees | 4s 4 4irs trees    |
| 14   | as a airs trees | 4s 4 4irs trees    |
| ...  | as a airs trees | 4s 4 4irs trees    |
| 21   | as a airs trees | as a airs tree5    |
| 22   | as a airs trees | as a air5 tree5    |
| 23   | as a airs trees | a5 a air5 tree5    |
| 24   | as a airs trees | a5 a air5 tree5    |
| ...  | as a airs trees | a5 a air5 tree5    |



参考：https://cloud.tencent.com/developer/article/1180351



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%AD%97%E5%85%B8%E7%94%9F%E6%88%90-pydictor/  

