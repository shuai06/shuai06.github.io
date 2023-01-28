# C语言琐碎复习（内容简单,仅自己看）



  
> 由于好久时间不看C了，好多东西都忘了
>
> 本文是跟着郝斌老师的视频做的随堂笔记，课下也没做整理，并不成系统，只是一些简单堆叠，且编码规范都没有，看看就好哈哈


  
#### 字符串

```c
char greeting[6] = {'H', 'e', 'l', 'l', 'o', '\0'};


//1	strcpy(s1, s2);
复制字符串 s2 到字符串 s1。
    
//2	strcat(s1, s2);
连接字符串 s2 到字符串 s1 的末尾。
    
//3	strlen(s1);
返回字符串 s1 的长度。
    
//4	strcmp(s1, s2);
如果 s1 和 s2 是相同的，则返回 0；如果 s1<s2 则返回小于 0；如果 s1>s2 则返回大于 0。
    
//5	strchr(s1, ch);
返回一个指针，指向字符串 s1 中字符 ch 的第一次出现的位置。
    
//6	strstr(s1, s2);
返回一个指针，指向字符串 s1 中字符串 s2 的第一次出现的位置。
   
    
//strlen 与 sizeof的区别：
strlen 是函数
sizeof 是运算操作符
二者得到的结果类型为 size_t，即 unsigned int 类型。

sizeof 计算的是变量的大小，不受字符 \0 影响；
strlen 计算的是字符串的长度，以 \0 作为长度判定依据。
    
   
// 'a' 表示是一个字符，"a" 表示一个字符串相当于 'a'+'\0';
    
    
//字符串遍历
char hi[] = "hello";
for(i==0, i<6,i++)
{
    printf("%c",hi[i]);
}
```



#### 运算符优先级nnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnhjjjjjjjjjjjjjjj

| **优先级** | **运算符**       | **名称或含义**            | **使用形式**              | **结合方向** | **说明**         |
| ---------- | ---------------- | ------------------------- | ------------------------- | ------------ | ---------------- |
| **1**      | []               | 数组下标                  | 数组名[常量表达式]        | 左到右       |                  |
| ()         | 圆括号           | （表达式）/函数名(形参表) |                           |              |                  |
| .          | 成员选择（对象） | 对象.成员名               |                           |              |                  |
| ->         | 成员选择（指针） | 对象指针->成员名          |                           |              |                  |
| **2**      | -                | 负号运算符                | -表达式                   | **右到左**   | 单目运算符       |
| (类型)     | 强制类型转换     | (数据类型)表达式          |                           |              |                  |
| ++         | 自增运算符       | ++变量名/变量名++         | 单目运算符                |              |                  |
| --         | 自减运算符       | --变量名/变量名--         | 单目运算符                |              |                  |
| *          | 取值运算符       | *指针变量                 | 单目运算符                |              |                  |
| &          | 取地址运算符     | &变量名                   | 单目运算符                |              |                  |
| !          | 逻辑非运算符     | !表达式                   | 单目运算符                |              |                  |
| ~          | 按位取反运算符   | ~表达式                   | 单目运算符                |              |                  |
| sizeof     | 长度运算符       | sizeof(表达式)            |                           |              |                  |
| **3**      | /                | 除                        | 表达式/表达式             | 左到右       | 双目运算符       |
| *          | 乘               | 表达式*表达式             | 双目运算符                |              |                  |
| %          | 余数（取模）     | 整型表达式/整型表达式     | 双目运算符                |              |                  |
| **4**      | +                | 加                        | 表达式+表达式             | 左到右       | 双目运算符       |
| -          | 减               | 表达式-表达式             | 双目运算符                |              |                  |
| **5**      | <<               | 左移                      | 变量<<表达式              | 左到右       | 双目运算符       |
| >>         | 右移             | 变量>>表达式              | 双目运算符                |              |                  |
| **6**      | >                | 大于                      | 表达式>表达式             | 左到右       | 双目运算符       |
| >=         | 大于等于         | 表达式>=表达式            | 双目运算符                |              |                  |
| <          | 小于             | 表达式<表达式             | 双目运算符                |              |                  |
| <=         | 小于等于         | 表达式<=表达式            | 双目运算符                |              |                  |
| **7**      | ==               | 等于                      | 表达式==表达式            | 左到右       | 双目运算符       |
| !=         | 不等于           | 表达式!= 表达式           | 双目运算符                |              |                  |
| **8**      | &                | 按位与                    | 表达式&表达式             | 左到右       | 双目运算符       |
| **9**      | ^                | 按位异或                  | 表达式^表达式             | 左到右       | 双目运算符       |
| **10**     | \|               | 按位或                    | 表达式\|表达式            | 左到右       | 双目运算符       |
| **11**     | &&               | 逻辑与                    | 表达式&&表达式            | 左到右       | 双目运算符       |
| **12**     | \|\|             | 逻辑或                    | 表达式\|\|表达式          | 左到右       | 双目运算符       |
| **13**     | ?:               | 条件运算符                | 表达式1? 表达式2: 表达式3 | 右到左       | 三目运算符       |
| **14**     | =                | 赋值运算符                | 变量=表达式               | **右到左**   |                  |
| /=         | 除后赋值         | 变量/=表达式              |                           |              |                  |
| *=         | 乘后赋值         | 变量*=表达式              |                           |              |                  |
| %=         | 取模后赋值       | 变量%=表达式              |                           |              |                  |
| +=         | 加后赋值         | 变量+=表达式              |                           |              |                  |
| -=         | 减后赋值         | 变量-=表达式              |                           |              |                  |
| <<=        | 左移后赋值       | 变量<<=表达式             |                           |              |                  |
| >>=        | 右移后赋值       | 变量>>=表达式             |                           |              |                  |
| &=         | 按位与后赋值     | 变量&=表达式              |                           |              |                  |
| ^=         | 按位异或后赋值   | 变量^=表达式              |                           |              |                  |
| \|=        | 按位或后赋值     | 变量\|=表达式             |                           |              |                  |
| **15**     | ,                | 逗号运算符                | 表达式,表达式,…           | 左到右       | 从左向右顺序运算 |

> 注：同一优先级的运算符，运算次序由结合方向所决定。



#### 数组

main.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 2.二维数组

/**
int a[3][4];
总共12个元素，可以当做3行4列看待，依次是:
a[0][0],a[0][1],a[0][2],a[0][3]
a[1][0],a[1][1],a[1][2],a[1][3]
a[2][0],a[2][1],a[2][2],a[2][3]

a[i][j]  ==> i+1行，j+1列的元素

int a[m][n];  该二位数组的右下角位置的元素只能是a[m-1][n-1]

*/

void init_double()
{
	//1.最笨的初始化
	int a[3][4]={1,2,3,4,5,6,7,8,9,10,11,12};

	//2.分开写（格式要写正确）
	int b[3][4]={
		{1,2,3,4},
		{5,6,7,8},
		{9,10,11,12}
	};
	
	//3.其他各种初始化



	// 输出二维数组内容
	int i, j;
	for(i=0;i<3;++i)
	{
		for(j=0; j<4; ++j)
		{
			printf("%-5d \t", b[i][j]);    // 付好：左对齐，5代表占5个位置（这不是重点，格式控制有其他控制办法）
		}
		printf("\n");  // 格式化(每四个一行)
	}


	// 二维数组排序
	//求每一行最大值
	//

}


// 多维数组：
/*
1. 不存在! 因为内存是线性一维的
2. n维数组可以当做 每个元素是n-1维数组的一维数组 的数组
比如：a[3][4]，可以当做含有三个元素的一维数组，只不过每个元素都可以划分为4个小元素

*/


```



#### 函数

func.h

```c
#include <stdio.h>

int after_func();   // 函数的声明， 可以不写变量名
void get_max(int, int);  // 函数声明，记得加分号
```

main.c

```c
#include "func.h"


void before_func()
{
	printf("我是函数测试，在主函数之前定义的。");
	getchar();
}


void main()
{

// 函数定义写在主函数之前的时候，那么就无需在主函数里面声明这个函数，只需要把函数体写在主函数前即可
//否则，需要先声明.

	// 此函数定义在主函数之前，直接调用即可，无须声明
	//before_func();

	//在主函数之后定义的，需要先声明才能调用。
	//1.声明(要放在main外面，我放在了自定义的头文件里面)
	//2.调用
	//after_func();

	// 求最大值
	//int i,j;
	//i=5;j=10;
	//get_max(i, j);
	int i;
	for(i=0;i<20;++i)
	{
		get_max(i,i+1);
	}
	getchar();

}


int after_func()
{
	printf("我是函数测试，在主函数之后定义的，需要先声明才能调用。");
	getchar();
	return 0;
}


//求最大值
void get_max(int a, int b)
{
	int max_val = a>b?a:b;

	printf("%d和%d比较，最大值为: %d \n", a,b,max_val);
}


/*
return和break的区别：
1.break:循环、switch
2.return终止函数

*/


/*
函数的分类：
1.有参函数和无参函数
2.有返回值和无返回值
3.库函数和用户自定义函数
4.普通函数和主函数(main)：一个程序只能有一个主函数，主函数既是程序的入口，也是程序的出口
主函数能调用主函数，其他函数不能调用main函数，普通函数可以相互调用
5.值传递和引用传递(其实这样说是不对的)

*/

/*
如何在软件开发中，合理的设计函数来解决实际问题：
功能划分的详细一些，提高利用率
函数是C语言的基本单位，类是Java，C#等的基本单位
*/


/*
常用的系统函数:
1. doulbe sqrt(double x)           求x的平方根
2. int abs(int x)				   求整数的绝对值
3. double fabs(double x)	       			
推荐书籍：
机械工业出版社：《turboc2.0使用大全》
*/


/*
递归：
栈：先进先出
*/


/*
变量的作用于和存储方式：
1.按作用域分：全局变量，局部变量（函数内部定义的变量/函数的形参）
2.按变量的存储方式：静态变量，自动变量，寄存器变量

*/

int k=99;

void test_var()
{
	//局部变量与全局变量命名相同：局部变量会屏蔽掉全局变量

}

```





#### 指针

test.h

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>


#define N 5

void test1();
void test2();
void test3();
void test4();
void test5();
void test6();
void test7();
void test8();
void base_pointer();
void free_test();
void exchange_two();
void test_exchange1(int,int);
void test_exchange2(int,int);
void test_exchange3(int,int);
void two_pointer();
void dynami_memo1();
void dynami_memo2();
void dynami_memo3();
void dynami_memo4();
void dynami_memo5();
void dynami_memo6();
```

main.c

```c
#include "test.h"

void main()
{
	//base_pointer();
	exchange_two();
}

void test1()
{
	int j=3;
	int * q; // q是变量名, int *表示q变量存放的是int类型变量的地址（int * 是数据类型）
	// 错误写法：q=55;   后面不能是值，只能跟地址
	q=&j;   //*q与j是等同的
	/*
	1.q保存了j的地址，引起：q指向j。
	2.q不是j，j不是q。修改q的值不会影响j,修改j的值也不会影响q的值
	3.如果一个指针变量指向一个普通变量，则*指针变量完全等同于普通变量
	
	*/

	/*
	指针：就是地址，地址就是指针
	指针变量：存放地址的变量
	两者不是一个概念
	但是，通常在叙述中，会把指针变量简称为指针
	*/

	// 一级指针
    // 指针的本质： 存地址(内存单元的编号)，间接访问
    // 指针使用场景：

    // & 取地址操作符/【引用】
    // * 取值操作符/【解引用】

}

void test2()
{
	/*
	指针的重要性;
	表示一些复杂的数据结构;
	快速的传递数据
	使函数返回一个以上的值
	能直接访问硬件
	能方便的处理字符串
	是理解面向对象语言中引用的基础
	*/

	//地址：
	/*
		内存单元的编号
		从零开始的非负整数
		范围：
	*/
    
	//指针：
	/*
	指针就是地址，地址就是指针
	指针变量就是存放内存单元的变量，
	两者不是一个概念
	但是，通常在叙述中，会把指针变量简称为指针
	指针的本质就是一个操作受限的非负整数
	
	*/

	//指针的分类：
	/*
	1.基本类型的指针
	int *p;
	int i=10;
	int j;
	p = &i;
	i=*p;
	printf("%d",j);
	
	例子，见函数base_pointer
	*/
}

void base_pointer()
{
	int *p;
	int *q;
	int i=5;

	//*p = i; //【错误】，*p代表的是p指向的地址的变量的值，p现在指向的垃圾值(还没有初始化)
	//应该这样
	p = &i;

	//*q=p;  //【错误】，有语法错误，*q是整形，p是int*类型
	//*q =  *p; //【错误】,语法没错，但是q指向垃圾值（没初始化）
	//应该这样：
	//p=q;  // 【错误】，q赋给p，p也变成垃圾值
	//printf("%d\n", *p);  // q的空间是属于本程序的，所以本程序可以读写q的内容，但是如果q的内部是垃圾值，则本程序不能读写*p的内容,因为*q所代表的的内存单元的控制权限并没有分配给本程序
	//

	// 指针变量的运算：
	// 指针变量不能相加、相乘、相除（能相减：如果两个指针变量指向的是【同一块连续空间中的不同存储单元】）
	/*
		p = &a[1];
		q = &a[3];
		printf("%d", q-p);
	*/

	getchar();

}


//指针的偏移
void test3()
{
    int a[N]={1,2,3,4,5};
    int *p;
    int i;
    p = &a[4];  // 指向a最后一个元素

    for(i=0; i<N; i++)     // 指针偏移
    {
        printf("%3d", *(p-i));     // p +sizeof(int)
    }
    printf("\n");

	getchar();
}

//指针与自增
void test4()
{
    int a[3]={2,7,8};
    int *p;
    int j;

    p = a;
    j = *p++;     // j = *p; p=p+1。     
    printf("a[0]=%d, j=%d, *p=%d \n", a[0], j, *p);  // 2,2,7

    j = p[0]++; // j=p[0],  p[0] = p[0] +1
    printf("a[0]=%d, j=%d, *p=%d \n", a[0], j, *p);  // 2,7,8  ？？？  

	getchar();
}

//野指针: free以后的指针未被赋值为NULL  (指向了一个未知的空间)
void test5()
{
	
    int *p, *p1, *p2, *p3;
    p = (int *)malloc(4);
    *p = 1;
    free(p);

    p = NULL;   // 赋值null以后，后面用的话会崩溃，能迅速找到问题

    p1 = (int *)malloc(4);  
    *p1 = 2;  // 没有free

    p2 = (int *)malloc(4);
    *p2 = 2;
    free(p1);

    p3 = (int *)malloc(4);
    *p3 = 3;
    *p1 = 20; // 以为p1free了，这里又赋值
    printf("p3 = %d", *p3);   //  20了    malloc有缓存机制
}



// 经典程序：互换两个数字
void exchange_two()
{
	int a=3;
	int b=5;
	//int t;

	//之前的做法
	//t=a;
	//a=b;
	//b=t;

	// 现在要求调用一个函数实现
	test_exchange3(&a, &b);

	printf("a=%d,b=%d",a,b);

	getchar();

}

// 不能完成互换功能
void test_exchange1(int a, int b)
{
	int t;

	t=a;
	a=b;
	b=t;
}


// 不能完成互换功能，只是把q和p的值换了，a和b并没有互换
void test_exchange2(int *p, int *q)
{
	int *t; // 如果要互换，变量类型必须一致	
	t = p; 
	p = q;
	q = t;

	// 形参的改变，不会改变实参的值
}


// 可以完成互换: p是a的地址，*p就是a
void test_exchange3(int *p, int *q)
{
	int t; // 如果要互换，变量类型必须一致	
	t = *p;  // *p是int，p是int* 
	*p = *q;
	*q = t;
}


// *的含义
void test6()
{
	//1.乘法
	//2.定义指针变量 
	int *p;  //定义了一个叫p的变，int *表示p只能存放地址
	//3.指针运算符：取地址的逆运算/间接访问
	// 放在已经定义好的指针变量的前面，取改指针的值
	//*p 表示以p的内容为地址的变量

	//写法：下面三种是等价的
	//int * p;
	//int *p;
	//int* p;
}

void free_test()
{
/*
脏数据
申请的空间不释放：内存泄露
堆空间：一旦申请，一直占用，除非free

*/
    int i;
    char *p;
    scanf("%d", &i);
    p = (char *)malloc(i);
    strcpy(p, "hello");   // p指向堆空间
    // p = "hellp";
    puts(p);
    free(p);  // 释放
    printf("free p ~~~~~~~~~~~");
}


void test7()
{
	// 一个指针变量到底占几个字节？
	/*
	指向不同类型，是不同的，比如int 一般是4个字节, char就是1个。     sizeof()可以查看：sizeof(double)
	p,q,r所占的字节数是一样的
	【总结：】一个指针变量，无论他指向的变量占几个字节，该指针变量只占四个字节。
	一个变量的地址使用该变量首字节的地址来表示
	*/
}


//动态内存分配（重点）
void test8()
{
	/*
	1.传统数组的缺点
		a.数组长度必须事先制定，且只能是长整数，不能是变量（int len=5; int a[len]; 这样是错误的）
		b.传统形式定义的数组，该数组的内存，程序员无法手动释放（一旦定义，该分配空间一直存在，直到数组所在的函数运行结束，数组的空间才会被系统释放）
		c.数组的长度不能再函数运行过程中动态扩充或缩小(长度一旦定义，其长度就不能再更改)
		d.A函数定义的数组，在A函数运行期间可以被其他函数使用；但A函数执行完毕后，A函数中的数组将无法再被其他函数使用
		传统数组也叫静态数组
			
	2.为什么需要动态分配内存
		解决了传统数组的4个缺陷

	3.动态内存分配举例_  动态数组的构造

	4.静态内存和动态内存的比较

	5.跨函数使用内存的问题
	
	*/

}

void dynami_memo1()
{
	// malloc 是memory(内存) allocate(分配)的缩写

	int i = 5;
    char *p;
    p = (char *)malloc(i);  
	/*
	
		1.要使用malloc，必须添加malloc.h或stdlib.h头文件
		2.malloc只有一个形参，且形参必须是整数类型
		3.上面，表示请求系统为本程序分配5个字节
		4.malloc函数只能返回第一个字节的地址，所以前面要把地址强制转换为int*
		5.p本身的内存是静态分配的，p所指向的内存是动态分配的
	*/

    strcpy(p, "hello");   // p指向堆空间
    // p = h e l l o \o;
    puts(p);
    free(p);  // 释放p所指向的内存 （p本身的内存是静态的，不能由程序员手动释放）
    printf("free p ~~~~~~~~~~~");
// 脏数据
// 申请的空间不释放：内存泄露
// 堆空间：一旦申请，一直占用，除非free
}

void f(int *q)
{
	// *p = 200; // error
	// q = 20   //error
	// q是p的一份拷贝(副本)
	// **q=200  // error
	*q=200;
}

void dynami_memo2()
{
	//int *arr = (int*)malloc(20);
 //   int *p;
 //   arr[0] = 1;
 //   arr[1] = 1;
 //   arr[2] = 1;
 //   arr[3] = 1;

 //   p = (int*)realloc(arr, 40);    // 扩展
 //   p[4] = 6;

	int *p = (int *)malloc(sizeof(int));
	*p = 10;

	printf("%d\n", *p); //10
	f(p);
	printf("%d\n", *p); //200
}

//动态内存分配举例 动态一维数组的构造
void dynami_memo3()
{
	int len;
	int *pArr;
	int i;

	printf("请输出要存放元素的个数：");
	scanf("%d", &len); // 5
	pArr = (int *)malloc(4*len);  // 数组长度为len； 5*4=20,（默认pArr指向第一个字节，但是由于进行了int强转，所以pArr指向前四个字节,pArr指向后面4个字节）
	//类似于 int pArr[len];

	//赋值
	for (i=0; i<len; ++i)
	{
		scanf("%d", &pArr[i]);
	}
	
	//输出
	printf("一维数组的内容是: \n");
	for (i=0; i<len; ++i)
	{
		printf("%d, ", pArr[i]);
	}

	realloc(pArr, 40);    // 扩展

	//释放数组
	free(pArr);
}

//动态内存和静态内存的比较
void dynami_memo4()
{
	/*
	静态内存是由系统自动分配，由系统自动释放，是在栈分配的
	动态内存是由程序员手动分配，手动释放，是由堆分配的

	malloc是堆分配的
	栈是存储结构，堆是排序方式
	*/
}

void multi_test(int **pp)
{
	int j=300;
	//  pp是p的地址，那*pp 就是p
	*pp = &j;   // 修改
	**pp = 200; // 修改

}
//多级指针（意义：跨函数使用内存）
void two_pointer()
{
	// 【场景】：当在子函数中需要修改主函数某一个一级指针变量的值，就需要二级指针
    //char b[N][10] = {"lilei","hanmeimei","xiyangyang","huitailang", "xiongda"};
    //char * p[N];  // 指针数组，里面存储的指针类型就是 二级指针   // &p[0]        每次偏移是固定的，64位数下就是8个字节
    //// char **p2;
    //int i;
    //for(i=0; i<N; i++)
    //{
    //    p[i] = b[i];

    //}

    //for(i=0; i<N; i++)
    //{
    //    puts(p[i]);

    //}

	int i = 10;
	int *p = &i;
	int **q = &p;
	int ***r = &q;

	//r = &p;   // 错误，因为r是int**类型，r只能存放int**类型变量的地址
	printf("%d",***r); // r的值

	//在另一个函数内部改变变量值
	multi_test(&p);  // p是int *, 那么 &p是int**
}


void f1(int **q)
{
	int i=5;
	//*q等价于p，q和**q都不等价于p
	// *q=i // error
	*q = &i;  // p=&i
	//**q=i;
}

//静态变量不能跨函数使用
void dynami_memo5()
{
	int *p;

	f1(&p);
	printf("%d", *p);  // 本语句语法没有问题，但逻辑上有问题（因为f函数中i的静态分配的，f结束，i的空间就释放了）
}

void f2(int **q)
{
	*q = (int *)malloc(sizeof(int));
	// 等价于p = (int *)malloc(sizeof(int));
	//q=5;  // error
	//*q=5; // p=5   error
	**q=5; // *p=5
}

//动态内存可以跨函数使用（重点）
void dynami_memo6()
{
	int *p;
	f2(&p);
	printf("%d", *p); // 能改变。p指向的是上面堆里面分配的空间

}

```







#### 结构体

jiegouti.h

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>


void test();
void struc_func();
void InputStudent(struct Student *);
void OutputStudent(struct Student);
void OutputStudent2(struct Student *);
void test_union();

```

main.c

```c
#include "jiegouti.h"

void main()
{

}

void test()
{       
		/*
		为什么需要结构体？
			为了表示一些复杂的事物，而普通的数据类型无法满足实际需求


		结构体：
			把一些基本类型组合到一起，形成的一个新的符合数据类型


		大小：
			结构体：总大小一般情况下等于各成员大小之和(先不考虑内存对齐)
			共用体：大小等于成员中最大的那个大小。（通常和结构体一起使用）
		

		如何定义(推荐第一种)：
			1. 推荐（这只是定义了一个新的数据类型，并没有定义变量）
			 struct student{
				int num;
				char name[20];
				int age;
			 };

			2. 这样只能定义一次
			 struct student{
				int num;
				char name[20];
				int age;
			 } st1;

			 3. 类型都没写...
			 struct {
				int num;
				char name[20];
				int age;
			 } st1;

			 //赋值和初始化(定义同时可以整体赋值,定义玩之后只能单个赋值)
				 定义同时初始化： struct student s1 = {1,'xxx',18};
				 先定义，后赋值：
								  struct student s2;
								  s2.num=1;
								  s2.name='yyy';
								  s2.age=19;


			 //取出结构体变量中每一个成员【重点】
				1.结构体指针变量名.成员名
				2.指针变量名->成员名    （第二种更常用）
							struct student *p = &s;  //注意是&
							p->name; //第一种方式(指针变量->成员变量名,在计算机内部会被转换为(*指针变量名).成员名)
							(*p).name; //第二种方式，与 s.name; 等价


			//结构体变量的运算
				结构体变量不能相加，相减，也不能相互乘除。
				但结构体变量可以相互赋值

			//结构体变量和结构体变量指针作为函数参数传递的问题

			//动态构造结构体数组

			//链表
		
		*/

		int num;

	    struct student{
			int num;
			char name[20];
			char gender;
			int age;
			float score;
			char addr[30];
		} ss={1,'xx','nan',20,99.9,'hhh'};   // ss就是变量 可以直接在这里初始化

		//赋值和初始化
		struct student ss2 = {2,'yy','bei',20,99.9,'zzz'};
		struct student s = {100,'lala','M',20,99.5, 'beij'};


		// 结构体数组
		struct student sArr[3];

		for (int i = 0; i < 3; i++)
		{
			scanf("%d%s %c%d%f%s", &sArr[i].num, sArr[i].name, &sArr[i].gender, &sArr[i].age,&sArr[i].score, &sArr[i].addr); // %c 忽略空格
		}


		// 结构体指针

		struct student *p = &s;  //注意是&
		//p = &s;
		// 获取成员

		p->name; //第一种方式(指针变量->成员变量名,在计算机内部会被转换为(*指针变量名).成员名)
		(*p).name; //第二种方式，与 s.name; 等价

		// p->name的含义：p所指向的那个结构体变量中的name这个成员


		num = p->num++;   // num = p->num, num=p->num+1;   //优先级
		// 第一时间去掉++
		num = p++->num;    // 没问题, num=p->num,  p=p+1


		//浮点型默认在C语言中是double，如果要定义成float，则在后面加f或F， 但是不能准确存储

}

///////////////////////////////////////////

struct Student{
	int num;
	char name[20];
	int age;
};

// 通过函数完成对结构体变量的输入和输出
void struc_func()
{

	struct Student st;

	InputStudent(&st);  //对结构体变量输入
	OutputStudent(st);  //对结构体变量输出
	OutputStudent2(&st);  //对结构体变量输出


}


void InputStudent(struct Student * pstu) //pstu只占4个字节
{
	(*pstu).num=1;
	strcpy(pstu->name, "xxx");  
	pstu->age=18;

};

/*  这是无法改变主函数的值的
void InputStudent(struct Student stu)
{
	stu.num=1;
	strcpy(stu.name, "xxx");  
	stu.age=18;

};*/


////应该发送内容还是传送地址？
//如果传递的是内容，内存消耗大
void OutputStudent(struct Student stu)
{
	printf("%d, %s %d", stu.num,stu.name, stu.age);
}

// 如果传递的是地址，可能不安全，但是提高了速度，节省了内存（推荐这种方法）
void OutputStudent2(struct Student *pstu)
{
	printf("%d, %s %d", pstu->num,pstu->name, pstu->age);
}



/*
【指针优点大总结：】

快速的传递数据，较少了内存的消耗 【重点】
使函数返回一个以上的值  【重点】
能直接访问硬件
能方便的处理字符串


**/


//共用体
void test_union()
{
	//共用体：大小等于成员中最大的那个大小（通常和结构体一起使用）

	// union 共用体name
	// {
	//     成员表列
	// } 变量表列;

	union data{
		int i;
		char c;
		float f;
	}u1;

	// 共用体的起始地址和每个成员的起始地址是相同的
	// 一个时间内只能存取一个成员
    
	union data d; // 现在最大的就是4
	d.i = 4;
	d.c='A';
	d.f = 99.9;

}


```





#### typedef

```c
//1) 为基本数据类型定义新的类型名
typedef double REAL;

    
//2) 为自定义数据类型（结构体、共用体和枚举类型）定义简洁的类型名称
typedef struct tagPoint
{
    double x;
    double y;
    double z;
} Point;
/*
实际上完成了两个操作：
1、定义了一个新的结构类型
struct tagPoint
{
    double x;
    double y;
    double z;
} ;
2、使用 typedef 为这个新的结构起了一个别名，叫 Point
typedef struct tagPoint Point
*/


//3) 为数组定义简洁的类型名称
typedef char* PCHAR;
PCHAR pa
    

```





#### NULL

```
二进制全部为0的含义：  
    1.数值为0
    2.字符串结束标记符 '\0'
    3.空指针NULL
        NULL表示编号为0的地址
        NULL表示的是零，不是数字，而是代表内存单元的编号

        计算机规定了，以零为编号的存储单元的内存不可读、不可写
```





#### 文件

```c
//文件
void test_file()
{
	# define N 20

	FILE *fp;
    char c;
    char p;
    char ret;
    char buf[N] = {0};   // 先初始化
    // fp = fopen("file.txt", "r");
    fp = fopen("D:\Desktop\1.txt", "r+");    // argv[1] 是传递的第一个参数
    if(fp == NULL)   // 判断打开是否成功， -> 失败
    {
        perror("fopen");   // 定位函数执行错误原因（只能定位刚刚执行的）
        goto error;

    }
    // fgetc   读取，一次读取一个字节
   while ((c = fgetc(fp)) != EOF)
   {
       printf("%c",c);
   }
   

    // fputs   写入，一次写入一个字节
    p = 'H';
    ret = fputc(p, fp);


	// printf(FILE *fp,const char *format, ...) 函数来写把一个字符串写入到文件中
	fprintf(fp, "This is testing for fprintf...\n");

    

    // fread:      buf没有\0
    ret = fread(buf, sizeof(char), N, fp);
    printf("%d, %s \n", ret, buf);

    // fwrite:
    strcpy(buf, "word");
    ret = fwrite(buf, sizeof(char), strlen(buf), fp);

    // 清空buffer
    memset(buf,0,sizeof(buf));




    // fseek
    fseek(fp, 0, SEEK_SET);   //  移动到开头。	SEEK_SET,偏移量的起始点:文件开始处

    fseek(fp, -12, SEEK_CUR);   // 往回偏移, 与磁盘上实际的字节数对应。	SEEK_CUT:文件当前位置 
							    // SEEK_END,偏移量的起始点：SEEK_END


    // ftell      获取文件读写指针的当前位置
	long last = ftell(fp);


    fclose(fp);  

    error:
        printf("error!!!");
        // fclose(fp);  // 关闭文件
}

```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/c%E8%AF%AD%E8%A8%80%E7%90%90%E7%A2%8E%E5%A4%8D%E4%B9%A0%E5%86%85%E5%AE%B9%E7%AE%80%E5%8D%95-%E4%BB%85%E8%87%AA%E5%B7%B1%E7%9C%8B/  

