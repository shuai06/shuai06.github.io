# php变量覆盖




> 可以用自定义的参数值替换原有变量值的情况称为变量覆盖漏洞



## 场景

- $$使用不当
- extract()函数使用不当
- parse_str()函数使用不当
- import_request_variables()使用不当
- 开启了全局变量注册
- ... ... 





## 1. $$导致的变量覆盖

> 经常在foreach中出现



使用foreach来遍历数组中的值，然后再将获取到的数组键名作为变量，数组中的键值作为变量的值

```php
<?php
    
//?name=test
//output:string(4) “name” string(4) “test” string(4) “test” test

$name='thinking';

foreach ($_GET as $key => $value)
    $$key = $value;
    var_dump($key);
    var_dump($value);
    var_dump($$key);

echo $name;

?>

```

例题：

```php

<?php
include "flag.php";
$_403 = "Access Denied";
$_200 = "Welcome Admin";

if ($_SERVER["REQUEST_METHOD"] != "POST")

    die("BugsBunnyCTF is here :p…");

if ( !isset($_POST["flag"]) )
    die($_403);

foreach ($_GET as $key => $value)
	key = value;

foreach ($_POST as $key => $value)
    $$key = $value;

	if ( $_POST["flag"] !== $flag )
		die($_403);

	echo "This is your flag : ". $flag . "\n";

	die($_200);

?>
```







## 2. extract()函数导致的变量覆盖

**extract()** 该函数使用数组键名作为变量名，使用数组键值作为变量值。针对数组中的每个元素，将在当前符号表中创建对应的一个变量。 

**语法：** `extract(array,extract_rules,prefix)`

```bash
extract 有三种形式可能导致变量覆盖:
第一种是第二个参数为EXTR_OVERWRITE,他表示如果有冲突，覆盖原有的变量。
第二种情况是只传入第一个参数，默认为EXTR_OVERWRITE模式。
第三种是第二个参数为EXTR_IF_EXISTS，他表示在当前符号表中已有同名变量时，覆盖它们的值,其他的都不注册新变量.
```



例题1

```php
<?php
$flag = 'xxx';
extract($_GET);

if (isset($gift)) {
   $content = trim(file_get_contents($flag));
   if ($gift == $content) {
       echo 'hctf{…}';
   } else {
        echo 'Oh..';
   }
} 
?>

    
    
//GET请求 ?flag=&gift=，extract()会将$flag和$gift的值覆盖了，将变量的值设置为空或者不存在的文件就满足$gift == $content。
//payload:
?flag=&gift=
```



例题2

```php
<?php if ($_SERVER["REQUEST_METHOD"] == "POST") { ?>
      <?php
        extract($_POST);
        if ($pass == $thepassword_123) { ?>
            <div class=”alert alert-success”>
                <code><?php echo $theflag; ?></code>
            </div>
        <?php } ?>
    <?php } ?>
    
    
//extract($_POST)会将POST的数据中的键名和键值转换为相应的变量名和变量值，利用这个覆盖$pass和$thepassword_123变量的值，从而满足$pass == $thepassword_123这个条件。
    
// POST Payload:
//pass=&thepassword_123=    
```











## 3. parse_str函数导致的变量覆盖

parse_str() 函数用于把查询字符串解析到变量中，如果没有array 参数，则由该函数设置的变量将覆盖已存在的同名变量。 

语法：`parse_str(string,array)`



例题1：

```php
<?php
error_reporting(0);

if (empty($_GET['id'])) {
    show_source(__FILE__);
    die();

} else {
    include ('flag.php');
    $a = "www.OPENCTF.com";
    $id = $_GET['id'];
    @parse_str($id);
    if ($a[0] != 'QNKCDZO' && md5($a[0]) == md5('QNKCDZO')) {
        echo $flag;
    } else {
        exit('其实很简单其实并不难！');
    }
}
?> 
    
    
//0e123会被当做科学计数法，0 * 10 x 123。所以需要找到一个字符串md5后的结果是0e开头后面都是数字的，如，240610708，s878926199a  
//利用php弱语言特性,
    
//使用GET请求id=a[0]=240610708，这样会将a[0]的值覆盖为240610708，然后经过md5后得到
0e462097431906509019562988736854
与md5(‘QNKCDZO’)的结果0e830400451993494058024219903391比较都是0
所以相等，满足条件，得到flag。
最终PAYLOAD： 
GET DATA: 
?id=a[0]=s878926199a 
or 
?id=a[0]=240610708
```

















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/php%E5%8F%98%E9%87%8F%E8%A6%86%E7%9B%96/  

