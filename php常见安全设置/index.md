# PHP常见安全设置


  

## PHP.INI与WEB安全

  

### 四种防护

#### 1.magic_quotes_gpc

魔术引号：` 过滤转义`四种字符（），对于sql注入有过滤作用

**绕过：** 特定条件下可以尝试` 宽字节注入`     




#### 2.safe_mod

> 安全模式，官方自带的

**安全模式：**  禁止php中敏感函数，防御提权及后门调用，漏洞利用
一些敏感函数被禁用

有时候不开安全模式的原因：自己开发中也需要使用到某些敏感函数，不然影响正常应用



#### 3.open_basedir

启用之后： open_basedir=D:\phpstudy\PHPTutorial\WWW`
限制后门的访问指定目录（菜刀连上也看不到目录里面）

>没办法绕过



#### 4.disable_function

可`自定义禁用函数`：安全模式(safe_mod)升级版，可自定义函数禁用，更加灵活

```php
dl, exec, system, passthru, popen, proc_open, pcntl_exec, shell_exec, mail, imap_open, imap_mail, putenv, ini_set, apache_setenv, symlink, link
```

举例：

```php
disable_function = eval,system,fopen,fread,readdir,init_set    //比如禁用某指定函数的做法
// 可以直接封杀大马小马等后门disable_functiondisable_function
```

##### 绕过

disable_function的突破:

###### **方法1：** 

蚁剑及插件(绕过disable_function)使用, 加载插件
插件会保存到antData/plugin目录
现在好像已经不用自己下载这个插件了

>参考文章:  https://www.cnblogs.com/linuxsec/articles/10966675.html



###### **方法2：**

 针对php7.x:（不支持windows）
https://github.com/mm0r1/exploits/blob/master/php7-gc-bypass/exploit.php



###### **方法3：**

 脚本集合：
https://github.com/l3m0n/Bypass_Disable_functions_Shell
http://webshell8.com/down/phpwebshell.zip



###### **方法4：**

一些webshell自带绕过功能



#### 其他

目录权限
文件解析

... ...









---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/php%E5%B8%B8%E8%A7%81%E5%AE%89%E5%85%A8%E8%AE%BE%E7%BD%AE/  

