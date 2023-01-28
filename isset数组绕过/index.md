# isset数组绕过






#### 1.数组返回NULL绕过

```php
<?php 

$flag = "flag";
if (isset ($_GET['password'])) {
	if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE) 
		echo 'You password must be alphanumeric'; 
	else if (strpos ($_GET['password'], '--') !== FALSE) 
		die('Flag: ' . $flag); 
	else 
		echo 'Invalid password';
}

?>
```



**php不能处理数组**

根据 `if (strpos ($_GET['password'], '--') !== FALSE)`

知道password一定有`'--'`。开始构造` %2d `但是过不了，根据题目暗示和php特性可以传一个数组绕过判断

```bash
http://123.206.87.240:9009/19.php?password[]=sad%2d%2d
```



> strpos() 函数查找字符串在另一字符串中第一次出现的位置。
>
> **注释：**strpos() 函数对大小写敏感。



> ereg()函数用指定的模式搜索一个字符串中指定的字符串,如果匹配成功返回true,否则,则返回false。搜索字母的字符是大小写敏感的。



#### 2.

```php
<?php
$flag = "flag";
if (isset ($_GET['ctf'])) {
if (@ereg ("^[1-9]+$", $_GET['ctf']) === FALSE) //①
echo '必须输入数字才行';
else if (strpos ($_GET['ctf'], '#biubiubiu') !== FALSE) //②
die('Flag: '.$flag);
else
echo '骚年，继续努力吧啊~';
}
?>
```



**思路：**

`ereg()`只能处理字符串的，遇到数组做参数返回NULL，判断用的是 === ，其要求值与类型都要相同，而NULL跟FALSE类型是不同的,所以①判断if不成立，继续执行②，这时strpos函数遇到数组，也返回NULL，与FALSE类型不同，if条件成立，输出flag。
所以payload应该是`?ctf[]=任意字符`,甚至就是`?ctf[]=`











---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/isset%E6%95%B0%E7%BB%84%E7%BB%95%E8%BF%87/  

