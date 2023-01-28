# C语言做题记录(入门版)


#### 1.编写一个程序求三个整数数的最大值

```c
/*
1.编写一个程序求三个整数数的最大值
*/

#include <stdio.h>


int main()
{
	int i,j,k, max=0;
	printf("请输入三个整数：");
	scanf("%d %d %d", &i, &j, &k);

	max = (i>j?i:j)>k?(i>j?i:j):k;
	printf("最大值为：%d", max);

	//getchar();
	
}
```



#### 2.输入10个数,并输出最大的那一个数

```c
/*
2.输入10个数,并输出最大的那一个数
*/

void ten_max()
{
	int arr[10], max=0;
	printf("请输入10个整数：\n");
	for(int i=0; i<=9; i++)
	{
		scanf("%d", &arr[i]);
	}

	//处理
	for(int i=0; i<=9; i++)
	{
		if(arr[i] > max)
		{
			max = arr[i];
		}
	}

	printf("最大值为：%d", max);

}



int main()
{
	//three_max();
	ten_max();

	
}

```



#### 3.判断2000-2500年中的闰年,并输出

```c
3.判断2000-2500年中的闰年,并输出
*/
void get_leap_year()
{
	for(int i=2000; i<=2500; i++)
	{
		if(i %400 ==0 || (i % 100 != 0 && i % 4 == 0))
		{
			printf("%d 是闰年\n", i);
		}
	}
}



int main()
{
	//three_max();
	//ten_max();
	get_leap_year();

	
}


```



#### 4.编写一个程序求1+2+3+…+…+100的值

```c++
void sum_100()
{
	int sum=0;
	for(int i=1; i<=100; i++)
	{
		sum += i;
	}
	printf("%d", sum);

}


int main()
{
	//three_max();
	//ten_max();
	//get_leap_year();
	sum_100();
	
}


```



#### 5.编写一个程序求(1)-(1/2)+(1/3)……+(1/99)-(1/100)的值

```c
/*
5.编写一个程序求(1)-(1/2)+(1/3)……+(1/99)-(1/100)的值
*/

void sum_100_sub()
{
	float sign=1,sum=0, deno, tmp;
	for(deno=1; deno<=100; deno++)
	{
		tmp = sign*(1/deno);    //分数形式
		sum = sum +  tmp;			//加和，全面存储
		sign = (-1)*sign;  //换号
	}
	printf("%f \n", sum);
}


int main()
{
	//three_max();
	//ten_max();
	//get_leap_year();
	//sum_100();
	sum_100_sub();
}
```





#### 6.判断一个数能否同时被3和5整除

```c
/*
6.判断一个数能否同时被3和5整除
*/
void division_3_5()
{
	int num;
	printf("输入一个整数：");
	scanf("%d", &num);
	if (num % 3 == 0 && num % 5 == 0)
	{
		printf("能同时被5和3整除");
		
	}
	else
	{
		printf("不能");
	}
}




int main()
{
	//three_max();
	//ten_max();
	//get_leap_year();
	//sum_100();
	//sum_100_sub();
	division_3_5();
	
}
```





#### 7.将具有n个元素的数组a中数按照从小到大排序并从大到小输出

```c
7.将具有n个元素的数组a中数按照从小到大排序并从大到小输出
*/

void sort_arr()
{
	int sum=0, num, a[100], n, tmp;

	printf("输入n:");
	scanf("%d", &n);
	for(int i=0; i< n; i++)
	{
		scanf("%d", &a[i]);
	}

	printf("元素为; ");
	for(int i=0; i< n; i++)
	{
		printf("%d", a[i]);
	}

	//排序
	for(int i=0; i<n; i++)
	{
		for(int j=i+1; i<n; j++)  // a[j]代表a[i+1]
		{
			if(a[i] > a[j])
			{
				tmp = a[i];
				tmp = a[j];
				a[j] = a[i];  //交换a[i]和a[i+1]
			}
		}
		//
	
	}


	//输出
	printf("从小到大元素为; ");

	for(int i=n-1; i >= 0; i--)
	{
		printf("%d", a[i]);
	}


}



```



#### 8.编写一个程序将大写字母转换成小写字母

```c
/*
8.编写一个程序将大写字母转换成小写字母
*/
void big_to_small()
{
	char c1, c2;
	scanf("%c", &c1);
	c2 = c1 + 'a'-'A';
	printf("%c \n", c2);
}


int main()
{
	big_to_small();
}

```





#### 9.编写一个程序将小写字母转换成大写字母

```c
/*
9.编写一个程序将小写字母转换成大写字母
*/
void small_to_big()
{
	char c1, c2;
	scanf("%c", &c1);
	c2 = c1 + 'A'-'a';
	printf("%c \n", c2);
}




int main()
{
	small_to_big();
}

```



#### 10.输入字符,最终按小写输出

```c
/*
10.输入字符,最终按小写输出
*/
void all_to_small()
{
	char ch;
	printf("输入字符：");
	scanf("%c", &ch);
	ch = (ch >='A' && ch <= 'Z')?(ch + 'a'-'A'):ch;
	printf("输出: %c\n", ch);
}


int main()
{
	all_to_small();
}


```





#### 11.给出三角形边长a、b、c计算出面积

```c
void tri_11()
{
	double a=2,b=2,c=3,p,s;
  
    if(a+b>c && a+c>b && b+c>a) //判断是否可以构成三角形。
    {
        p=(a+b+c)/2;//计算半周长
        s=sqrt(p*(p-a)*(p-b)*(p-c));//套用海伦公式，计算面积
        printf("面积为%lf\n", s);//输出结果
    }
    else printf("无法构成三角形\n");

}

```



#### 12.给出三角形三个点坐标计算出面积

```c
void tri_12()
{
	Point p1, p2, p3;
	double a,b,c, area;
	p1.x = 1;
	p1.y = 2;
	p2.x = 1;
	p2.y = 3;
	p3.x = 2;
	p3.y = 2;

	a = sqrt((p1.x-p2.x)*(p1.x-p2.x) + (p1.y-p2.y)*(p1.y-p2.y));
	b = sqrt((p2.x-p3.x)*(p2.x-p3.x) + (p2.y-p3.y)*(p2.y-p3.y));
	c = sqrt((p1.x-p3.x)*(p1.x-p3.x) + (p1.y-p3.y)*(p1.y-p3.y));

	area = (a+b+c)/2;
	printf("%lf", area);
}
```



#### 13.请编程序将”China”译成密码

密码规律是:用原来的字母后面第4个字母代替原来的字母。例如:字母“A”后面第4个字母是“E”，用“E”代替”A”。因此，”China”应译为”Glmre”。请编一程序，用赋初值的方法使c1,c2,c3, c4,c5这5个变量的值分别为’C'，'h’,'，‘n', ‘a’，经过运算，使c1, c2, c3, c4, c5分别为’G’，‘I,‘m’,‘r'，‘e’。分别用putchar函数和 printf 函数输出这个5个字符

```c
void china_pass()
{
	char c1='C',c2='h',c3='i',c4='n',c5='a';
	c1+=4;
    c2+=4;
    c3+=4;
    c4+=4;
    c5+=4;

	printf("%c%c%c%c%c",c1,c2,c3,c4,c5);

}
```



#### 14.给出一个百分制成绩，要求输出成绩等级A,B,C,D,E。90分以上为A，80~90分为B,70~79分为C,60~69分为D,60分以下为E。

```c
//14.给出一个百分制成绩，要求输出成绩等级A,B,C,D,E。90分以上为A，80~90分为B,70~79分为C,60~69分为D,60分以下为E。
void grade_bank()
{
	int grade;
	printf("请输入成绩(0-100): ");
	scanf("%d", &grade);

	switch (grade/10)
	{
	case 10:
	case 9:
		printf("A");
		break;
	case 8:
		printf("B");
		break;
	case 7:
		printf("C");
		break;
	case 6:
		printf("D");
		break;
	default:
		printf("E");
		break;
	}
	

}
```



#### 15.输出2的N次幂:1,2,4,8,.一直到2^N

注:要求不使用库函数

```c
#define N 16

//15.输出2的N次幂:1,2,4,8,一直到2^N   注:要求不使用库函数
void get_2_pow()
{
	int num=1;
	for(int i=0; i<=N; i++)  // i记录当前指数
	{
		
		printf("2 的%d次方是: %d \n", i, num);
		num *=2;	
	}
}





```







#### 16.判断一个数是否为素数

```c
int m;  // 输入的整数 
int i;  // 循环次数
int k;  // m 的平方根 

printf("输入一个整数：");
scanf("%d",&m);

// 求平方根，注意sqrt()的参数为 double 类型，这里要强制转换m的类型 
k=(int)sqrt( (double)m );
for(i=2;i<=k;i++)
    if(m%i==0)
        break;

// 如果完成所有循环，那么m为素数
// 注意最后一次循环，会执行i++，此时 i=k+1，所以有i>k 
if(i>k)
    printf("%d是素数。\n",m);
else
    printf("%d不是素数。\n",m);
return 0;
```





#### 17.输出2-300间的素数

```c
#include <stdio.h>
#include <math.h>

//17.输出2-300间的素数
int main()
{
	int num, i, count=0, k;
	
	for(num=2; num<=300; num++)
	{
	    k = (int)sqrt((double)num);
		
		for(i=2; i<=k; i++)
		{
			if(num%i == 0)
			    break;
		}

	   if(i>k)
	       printf("%d是素数。\n",num);
       else
           printf("%d不是素数。\n",num);

	
	}
	return 0;

}

```





#### 18.求2+22+222+2222+……，共5项

```c
//18.求2+22+222+2222+……，共5项
void cal_222()
{
	int n = 2, sum=0;
	for(int i=0; i<=5; i++)
	{
		n = n*10 + 2;
		sum += n;
		
	}
	
	printf("%d \n", sum);
}


```





#### 19．编程求`1!+2!+3!+4!...n!`的值,n小于等于10

```c
//19.编程求`1!+2!+3!+4!...n!`的值,n小于等于10

void get_sum1()
{
	int sum=0, k, m=1;
	printf("请输入n:");
	scanf("%d", &k);
	for(int i=1; i<=10; i++)
	{
		for(int j=1; j<=i; j++)   //循环到指定值
		{
			m = m * j;  //阶乘的结构
		}
		sum += m;
		m = 1;
	}
	printf("到%d的阶乘之和为: %d", k, sum);
}

```







#### 20．自然底数e=2.718281828…，现编程求e

e 的计算公式为:e=1+1/1!+1/2!+1/3!+...

求当最后一项的值小于10^(-10)时结束

```c
void get_e()
{
	float e =1.0, n=1.0;
	int i=1;

	while (1/n > 1e-10)
	{
		e += 1/n;
		i++;
		n = i*n;

	}

	printf("e的值为：%f\n", e);

}
```





#### 21.求1000以内的所有回文素数。

任意的**整数**，当从左向右读与从右向左读是相同的，且为素数时,称为回文素数。

```c
int sushu(int i)
{
    int j;
    if(i<=1)
        return 0;
    if(i==2)
        return 1;
    for(j=2;j<i;j++)
    {
        if(i%j==0)
            return 0;
        else if(i != j+1)
            continue;
        else
            return 1;
    }
}

void test()
{
	int i;
    for(i=10;i<1000;i++)
	{
	if(sushu(i)==1)
	{
		if(i/100==0) //如果是两位数
		{
			if(i/10==i%10)
				printf("%5d",i);
			if(i%5==0)
				printf("\n");
		}
		else  //如果是三位数
			if(i/100==i%10)
				printf("%5d",i);
			if(i%5==0)
				printf("\n");
	
	}
	
	
	}
}

```



#### 22．在数组中插入元素

有一个已经排好序的数组。输一个数，要求按原来的规律将它插入数组中。

```c
//22.在数组中插入元素
void insert_arr(int *arr, int len, int index, int val)
{
	for(int j=len-1; j>= index-1; j--)
	{
		arr[j+1] = arr[j];
	}
	arr[index-1] = val;

}




int main()
{
	int a[5] = {1, 22, 3, 7, 9};
	for(int i=0; i<5; i++)
	{
		printf("%d, ", a[i]);
	
	}

	insert_arr(a, 5, 2, 99);  //在2的位置插入99

	for(int i=0; i<6; i++)
	{
		printf("%d, ", a[i]);
	
	}


}
```





#### 23．求序列:2/1，3/2，5/3，8/5，13/8，21/13...的前20项之和。

```c
//23.求序列:2/1，3/2，5/3，8/5，13/8，21/13...的前20项之和。
void sum_20()
{
	double x=2, y=1, tmp;
	double sum=0;

	for(int i=0; i<20; i++)
	{
		sum += x / y;

		tmp = y;
		y = x;
		x = tmp + x;

	}
	
	printf("%f\n", sum);

}

```





#### 24．使用二维数组将一个3×4的矩阵中所有元素的最大值及其下标获取

```c
#include <stdio.h>

int main()
{
   # 是两个{{
	int arr[3][4] = {[1,2,3,4},{5,6,7,8},{9,55,3,11]};
	int max_val = arr[0][0], max_i, max_j;

	for(int i=0; i<3; i++)
	{
		for(int j=0; j<4; j++)
		{
			if (arr[i][j] > max_val)
			{
				max_val = arr[i][j];
				max_i = i;
				max_j = j;
			}
		
		
		}

	
	}

	printf("最大值为:%d, 下标为:%d,%d\n", max_val, max_i, max_j);


	
	return 0;

}
```





#### 25．编写程序实现矩阵转置

转置是指:将一个矩阵的行与列调换

```c
//25．编写程序实现矩阵转置
void revert_arr()
{
	int a[3][4] = {{1,2,3,4},{5,6,7,8},{9,55,3,11}};
	int b[4][3];



	for(int i=0;i<3;i++) 
	{
		for(int j=0;j<4;j++) 
		{
			 b[j][i]=a[i][j];  //转置
		}
	}

	//输出
	for(int i=0;i<4;i++) 
    { 
        for(int j=0;j<3;j++) 
		{
			printf("%d  ",b[i][j]); 
            //printf("\n"); 
		
		}
    } 



}

```





#### 26．求3乘3矩阵对角线元素之和

3乘3矩阵对角线有两条，对于二维数组对角线上数组元素下标满足`i==j || i+j==2`

```c
void get_medi_sum()
{
	int a[3][3] = {{1,2,3},{5,6,7},{9,55,3}};
	int sum=0;

	for(int i=0;i<3;i++) 
	{
		for(int j=0;j<4;j++) 
		{
			 if(i==j || i+j==2)
			 {
				sum += a[i][j];
			 
			 }
		}
	}

	printf("%d", sum);



}

```



#### 27.求水仙花数

阿姆斯特朗数也就是俗称的水仙花数、是指一个三位数，其各位数字的立方和等于该致本身。例如:153=1^3+5^3+3^3，所以153就是一个水仙花数。求出所有的水仙花数。

```c
void shuixianhua()
{
	int ge, bai, shi;
	for(int i=100; i<=999; i++)
	{
		ge = i % 10;
		bai = i/100;
		shi = i%100 / 10;
		if((ge*ge*ge + shi*shi*shi + bai*bai*bai) == i)
		{
			printf("%d ", i);
		
		}
	
	}


}
```





#### 28.求出前20个Fibonacci数列

斐波那契数列从0和1开始，之后的每一项就由其前面两数相加得到。在数学上，斐波纳契数列以如下递归的方法定义:`F(0)=0`，`F(1)=1`，`Fn=F[n-1)+F(n-2)(n>=2，n属于N*)`，求出前20个 Fibonacci数列

```c
int fibonacci(int i)
{
	if(i==0)
	{
		return 0;
	}
	else if(i==1)
	{
		return 1;
	}
	else
	{
		return fibonacci(i-1) + fibonacci(i-2);
	}

}

int main()
{
		for(int i=1;i<=20;i++)
		{
			printf("%d \t", fibonacci(i));
		}

}
```



或者:

```c
void fabs()
{

	int arr[20] = {1, 1};

	for(int i=2; i<20; i++)
	{
		arr[i] = arr[i-1] + arr[i-2];
	
	}

	//输出
	for(int j=0; j<20; j++)
	{
		printf("%d\t", arr[j]);

		//控制格式
		if(j % 5 == 0) printf("\n");

	}

}
```



#### 29.生兔子问题

假设一对兔子的成熟期是一个月，即一个月可长成成兔，那么，如果`每对`成兔`每个月`都`生一对小兔`,一对新生的小兔`从第二个月起`就开始生兔子，试问从一对兔子开始繁殖，以后每个月会有多少对免
子?

```c
//循环方式实现兔子生小兔子问题
void rabbit_fabs()
{
	int month, tu1=1,tu2=1, tu3;
	printf("请输入第几个月数:\n");
	scanf("%d",&month);

	if(month ==1 || month == 2)
	{
        printf("有一对兔子");

	}
	else if(month > 2)
	{
		for(int i=3; i<=month; i++)
		{
			tu3 = tu2 + tu1;
			tu1 = tu2;
			tu2 = tu3;
		}
		printf("%d 月的兔子数为：%d\n",month,tu3);


	}

}



//递归方式：
int rabbit_fabs_digui_test(int m)
{
	if(m ==1 || m == 2)
		return 1;
	else
	{
		return rabbit_fabs_digui_test(m-1) + rabbit_fabs_digui_test(m-2);
	}

}


//递归方式实现兔子生小兔子问题
void rabbit_fabs_digui()
{


	int month;
	printf("请输入第几个月数:\n");
	scanf("%d",&month);
	printf("%d 月的兔子数为：%d\n",month,rabbit_fabs_digui_test(month));

}

```





#### 30.输入一行英文句子，统计单词数

```c
//其实就是统计空格数字(单词数=空格数+1)

void count_word()
{
	int word=0,count=0;
	char ch;

	//输入
	while ((ch=getchar()) != '\n')
	{
		if(ch == ' ')
		{
			word = 0; //如果遇到空格就置word为0
		}
		else if(word==0)
		{
			word = 1;
			count ++;
		}

	}

	printf("单词数为: %d \n", count);
}

```





#### 31.加密解密

在对一个指定的字符串加密之后，利用解密函数能够对密文解密，显示明文信息。

加密的方式是将字符串中每个字符加上它在字符串中的位置和一个偏移值5。

以字符串“mrsoft”为例，第一个字符“m”在字符串中的位置为0，那么它对应的密文是“'m'+0+5",即r.

```c
#include <string.h>

void encrypt1()
{
	int flag = 1, str_len=0, i, j;
	char text[128] = {'\0'}, encrypt_text[128]={'\0'}; //明文字符串、密文字符串
	
	while (true)
	{
		if(flag == 1) //1加密
		{
			printf("输入原字符串: ");
			scanf("%s", &text);
			str_len = strlen(text);
			for(i=0; i<str_len; i++)
			{
				encrypt_text[i] = text[i] + i + 5;
			}
			encrypt_text[i]='\0';
			printf("加密后的字符串：%s\n", encrypt_text);

		}

		else if(flag ==2) //解密
		{
			str_len = strlen(encrypt_text);
			for(i=0; i<str_len; i++)
			{
				text[i] = encrypt_text[i] - i - 5;
			
			}
			text[i]='\0';
			printf("字符串：%s", text);
		}

		printf("请输入要选择的功能(1加密，2解密)：");
		scanf("%d", &flag);
	}




}


```





#### 32.逆序输出该整数

将一个从键盘输入的整数存放到一个数组中，通过程序的运行按照数组中的顺序输出该整数,要求使用递归的方法解决问题

```c
//递归思想

#include <stdio.h>

int convert(char s[],int n)
{
    int i;
    // 如: 4567
    if((i=n/10)!=0)
        convert(s+1,i);

    *s=n%10+'0';

    return 0;

}

int main()
{

    int num;

    char str[10]=" ";

    printf("input integer data：");

    scanf("%d",&num);

    convert(str,num);

    printf("output string：\n");

    puts(str);

    return 0;

}
```





#### 33.将一个一维数组就地逆置

(就地逆置指不再开辟第二个数组空间)

```c
//数组两端对称元素相互交换
//33.将一个一维数组就地逆置
void reverse_arr_there()
{
	//可以这么来求数组长度：    int cnt = sizeof(a) / sizeof(a[0]);

	int arr[10]= {1,2,3,4,5,6,7,8,9,10};
	int i=0, j = 9, tmp;
	while (i<=j)
	{
		tmp = arr[i];
		arr[i] = arr[j];
		arr[j] = tmp;
		i++;
		j--;
	}
    
	for (i = 0; i < 10; i++)
	{
		printf("%d\t", arr[i]);
	}
}



```





#### 34.数据加密

某个公司采用公用电话传递数据,数据是四位的整数，在传递过程中是加密的,**加密规则**如下:每
位数字都加上5,然后用和除以10的余数代替该数字，再将第一位和第四位交换,第二位和第三位交换。

```c
//每位数字都加上5,然后用和除以10的余数代替该数字，再将第一位和第四位交换,第二位和第三位交换。
void crypt_tel()
{
	int num, a[4], tmp;
	scanf("%d", &num);

	//个位 1 2 3 4
	a[0] = num % 10;
	//十位
	a[1] = num %100 / 10;
	//百位
	a[2] = num %1000 / 100;
	//千位
	a[3] = num / 1000;

	//加密
	for (int i = 0; i < 4; i++)
	{
		a[i] = a[i] + 5;
		a[i] = a[i]%10;
	}
    //交换
	for (int i = 0; i < 3/2; i++)
	{
		tmp = a[i];
		a[i] = a[3-i];
		a[3-i] = a[i];

	}

	//输出
	for (int i = 0; i < 4; i++)
	{
		printf("%d", a[i]);

	}

}

```





#### 35.输出菱形

```c
/*
设菱形的总行数为line，总列数为column，当前行为i，当前列为j。上半部分与下半部分的规律不一样，应该分开讨论。

我们着眼于星号(*)，思考什么条件下输出星号，总结出如下的规律。

1) 对于上半部分(包括中间一行)，当前行与当前列满足如下关系输出星号：
j>=(column+1)/2-(i-1)     (column+1)/2-(i-1)为第i行最左边的星号
j<=(column+1)/2+(i-1)    (column+1)/2+(i-1)为第i行最右边的星号

2) 对于下半部分，当前行与当前列满足如下关系输出星号：
j>=(column+1)/2-(line-i)     (column+1)/2-(line-i)为第i行最左边的星号
j<=(column+1)/2+(line-i)    (column+1)/2+(line-i)为第i行最右边的星号

不满足上述条件，则输出空格。
*/

void print_diamond()
{
	int line;  // 菱形总行数
    int column;  // 菱形总列数
    int i;  // 当前行
    int j;  // 当前列

	printf("请输入菱形的行数(奇数)：");
    scanf("%d", &line);
    if(line%2==0){  // 判断是否是奇数
        printf("必须输入奇数！\n");
        exit(1);
    }
    column = line;  // 总行数和总列数相同

	for(i=1; i<=line; i++){  // 遍历所有行

		if(i<(line+1)/2+1){  // 上半部分（包括中间一行）
			for(j=1; j<=column; j++){  // 遍历上半部分的所有列
				if( (column+1)/2-(i-1)<=j && j<=(column+1)/2+(i-1) ){
					printf("*");
				}else{
					printf(" ");
				}
			}
		}

		else{  // 下半部分
			for(j=1; j<=column; j++){  // 遍历下半部分的所有列
				if( (column+1)/2-(line-i)<=j && j<=(column+1)/2+(line-i) ){
					printf("*");
				}else{
					printf(" ");
				}
			}
		}
		printf("\n");
}
}
```





#### 36.求2-300范围内完数的个数。

(如果**一个数等于它的因子之和**，则称该数为“完数”(或“完全数”)。例如，6**的因子为**1、23,而6=1+2+3，因此6是“完数”）

```c
//36.2-300范围内完数的个数
void perfect_num()
{
	int i,j,sum;
	for (i = 2; i <= 300; i++) //外层控制取值范围
	{
		sum = 0;
		for (j = 1; j < i; j++)  //判断因子
		{
			if(i%j == 0) //能被整余，则为其中一个因子
			{
				sum += j;
			}

		}

		//判断因子之和与当前原数是否相同
		if(sum == i)
		{
			printf("%d是完数\n", i);
		}
		//else
		//{
		//	printf("%d不是完数\n", i);
		//}

	}

}


```



#### 37.求孪生素数

所谓李生素数指的是间隔为2的两个相邻素数，因为它们之间的距离已经近的不能再近了,如同李生兄弟一样,所以将这一对素数称为孪生素数。编程求出3~1000以内的所有孪生套数。

```c
//即i是素数，i+2(相邻的)也是素数


//37.求孪生素数
int twin_prime(int n)
{
	long m;
	m = sqrt((double)n);
	for(int i=2; i<=m; i++)
	{
		if(n%i == 0)  //能被除尽，不是素数
		{
			return 0;
		}
	}
	return 1;  //是素数
}

void twin_prime_test()
{
	int count;

	for (int i = 3; i <= 1000; i++)
	{
		if(twin_prime(i) && twin_prime(i+2))
		{
		
			printf("%-3d, %3d", i, i+2);

			//控制格式
			count ++;
			if(count % 5 == 0)
			{
				printf("\n");
			}

		}

	}
	

}


int main()
{
	twin_prime_test();
}

```





#### 38.求回文数

打印所有不超过n(取 n<256)的其平方的具有对称性质的装(也称回文数)

```c
//38.求回文数
//打印所有不超过n(取 n<256)的其平方具有对称性质的装(也称回文数)
void huiwen()
{
	//http://c.biancheng.net/view/505.html


	int m[16], n, t, i, count=0;

	long unsigned a,k;
	    printf("No.    number     it's square(palindrome)\n");
		for (n=1; n<256; n++) //穷举n的取值范围
		{
		    k=0; t=1; a=n*n;  /*计算n的平方*/
			for(i=0; a!= 0; i++) /*从低到高分解数a的每一位存于数组m[1]~m[16]*/
			{
				m[i] = a % 10;
				a /= 10;
			}

			for(; i>0; i--)
			{
				k += m[i-1] * t;
				t *= 10;
			}

			if(k == n*n)
            printf("%2d%10d%10d\n", ++count, n, n*n);
		}
}
```





#### 39.判断一个五位数是否是回文数

```c
void is_huiwen()
{
	int n;
	int ge, shi,bai, qian, wan;
	printf("输入一个数：");  // 65432
	scanf("%d", &n);
	wan = n /10000;
	qian = n /1000 %10;
	shi = n % 100 /10;
	ge = n % 10;
	printf("%d", wan);
	printf("%d", qian);
	printf("%d", shi);
    printf("%d", ge);

	if(ge == wan && shi == bai)
	{
		printf("%d是回文数\n", n);
	
	}
	else
	{
	    printf("%d不是回文数\n", n);
	
	}
}

```



#### 40.求100以内的所有勾股数。

所谓勾股数,是指能够构成直角三角形三条边的三个正整数（a, b. c).



采用穷举法求解时，最容易想到的一种方法是利用3个循环语句分别控制变最a、b、c的取值范围，第1层控制变量a，取值范围是1〜100。在a值确定的情况下再确定b值，即第2层控制变量b，为了避免结果有重复现象，b的取值范围是a+1〜100。a、b的值已确定，利用穷举法在b+1〜100范围内一个一个的去比较，看当前c值是否满足条件 a2 + b2 = c2，若满足，则输出当前a、b、c的值，否则继续寻找。

但是上述算法的效率比较低，根据 a2 + b2 = c2 这个条件，在a、b值确定的情况下，没必要再利用循环一个一个去寻找c值。若a、b、c是一组勾股数，则 a2 + b2 的平方根一定等于c，c的平方应该等于a、b的平方和，所以可将的平方根赋给c，再判断c的平方是否等于。根据“勾股数”定义将变量定义为整型，a2 + b2 的平方根不一定为整数， 但变量c的类型为整型，将一个实数赋给一个整型变量时，可将实数强制转换为整型（舍弃小数点之后的部分）然后再赋值，这种情况下得到的c的平方与原来的的值肯定不相等，所以可利用这一条件进行判断。

```c
void gougu()
{
	int a, b, c, count=0;
    printf("100以内的勾股数有：\n");
	for(a=1; a<=100; a++)
	{
		for(b=a+1; b<=100; b++)
		{
			c = (int)sqrt((double)(a*a + b*b));  //求c的值
			if(c*c==a*a+b*b && a+b>c && a+c>b && b+c>a && c<=100)  /*判断c的平方是否等于a2+b2*/
			{
				printf("%4d %4d %4d", a,b,c);
				count++;
				if (count % 5 == 0) //每五组就换行
					printf("\n");
			}
		}	
	}
}

```



#### 41.输入一个字符串,判断其是否为回文

所谓**回文字符串**,是指从左到右读和从右到左读完全相同的字符串。

```c
//从中间为界限，前后对应位置比较，如果相同，则为回文
  
void str_huiwen()
{
	char s[100];  // 存放输入的字符串 
    int i, j, n;
    printf("输入字符串：");
    gets(s);
    
    n=strlen(s);
    for(i=0,j=n-1;i<j;i++,j--)
	{
		if(s[i]!=s[j]) break;
	}
    
    if(i>=j)
        printf("是回文串\n");
    else
        printf("不是回文串\n");
}
    
```



#### 42删除字符串中间的*号

现在有一串字符需要输入，规定输入的字符串中只包含字母和`*`号。请编写程序，实现以下功能:除了字符串前后的`*`号之外,将串中其他的`*`号全部删除。
例如，假设输入的字符串为·\*\*\*A\*BC\*DEF\*G\*\*\*\*\*\*，删除串中的*号后，字符串变为\*\*\*ABCDEFG\*\*\*\*\*\*

```c
//可以设两个变量i j让i从字符串左边起往右移动，到第一个不为*的位置记下此时i的值，设j从字符串右边往左移动，到第一个不为星号的位置记下此时j的值.
//再设一个临时变量k,从i到j移动，遇到星号将其置0处理，然后输出即可


void delete_xing()
{
	char c[100] = {0};
	scanf("%s", c);
	int len = strlen(c), i, j;

	for (i = 0; i < len; i++)
	{
		if(c[i] != '*' && c[i-1] == '*')
			break;  //i从串左边起往右移动，第一个不为*的位置记下此时i的值
	}

	for (j = len-1; j >= 0; j--)
	{
		//j从字符串右边往左移动，到第一个不为星号的位置记下此时j的值
		if(c[j] != '*' && c[j+1] == '*')
		    break;  //第一个不为*的位置记下此时i的值
	}

	// 再设一个临时变量k,从i到j移动，遇到星号将其置0处理，然后输出即可
	for (int k = i; k <=j; k++)
	{
		if(c[k] == '*')
			c[k] = '0';
	}

	// 输出
	for(int k=0; k<len; k++)
		if(c[k]!='0')
			printf("%c",c[k]);


}


```



#### 43.实现函数strlen

C库函数strlen可以求string所指字符串串长，其函数定义:`int strlen (char *string)`.请根据上述函数定义,实现此函数。

```c
int strlen(char *string)
{
	int length=0;
	while (*string++ != '\0')
	{
		length++;
	}
	return length;

}

//测试函数
void strlen_test()
{
	char s[10] = "\0";
	int len;
	printf("请输入字符串：");

	gets(s);
	printf("%s\n", s);

	len = strlen(s);

	printf("%d\n", len);

}

```





#### 44.实现函数atoi

C库函数atoi可以实现字符串向整数的转换，其函数定义:`int atoi( char *s)`，请根据上述函数定义,实现此语数。

```c
//考点：字符串转换为数字时，对相关ASCII码的理解。
// a - '0'
#include <stdio.h>
#include <ctype.h>

int atoi(char *s)
{
	int num_val, flag, i;

	if(s == NULL)
	{
		return 0;
	}

	for (i = 0; i < isspace(i); i++) //直到空格前一直循环
	{
		flag = (s[i]=='-')?-1:1;

	}

	if(s[i] == '-' || s[i] == '+') //跳过运算符
		i++;

	//从数字开始计算
	for(num_val=0; isdigit(s[i]); i++)
		num_val = 10*num_val +(s[i] - '0');  //将数字字符转换成整形数字



	return num_val;
	
}

void atoi_test()
{
	char s[10] = "\0";
	int len;
	printf("请输入整数形式的字符串："); // -5678
	scanf("%s", s);
	
	printf("整数为: %d\n", atoi(s));


}

```





#### 45.实现函数itoa

C库函数itoa可以实现整数字向符串的转换,其函数定义:`void itoa (int n, char *s)`，请根据上述函数定义,实现此函数。

```c
//通过把整数的各位上的数字加“0”转换成char类型并存到字符数组中。但是要注意，需要采用字符串逆序的方法
// a + '0'


//45.实现函数itoa
void itoa(int num, char *s)
{
	int i,j,sign;
	//sign用来记录符号
	if((sign=num) < 0) 
		num = -num; //如果为负数，乘以-1来变为正数

	i=0;
	do{
		// 如： '5678'
		s[i++] = num%10 + '0';  //取下一个数字

	}while((num/=10) > 0);  //删除该数字     5678 --> 567 --> 56 --> 5
	
	if(sign < 0)
		s[i++] = '-';  //如果为负数，把减号放到数组末尾
	s[i] = '\0';

	//因为生成的数字是逆序的，所以要逆序输出
	for (j=i; j>=0; i=j--)
	{
		printf("%c", s[j]);
	}

}

void itoa_test()
{
	int num;
	char s[100];
	printf("Input num:");

	scanf("%d",&num);
	printf("the string : \n");

	itoa (num,s);
}
```



#### 46.实现函数 strcat

C库函数 strcat可以实现两字符串的连接,其函数定义:`char *strcat (char *strDes,const char  *strSrc)`.请根据上述函数定义,实现此函数。

```c
char *strcat (char *strDes,const char  *strSrc)
{
	char *tmp = strDes; //赋值首地址，为了最后方便返回

	while (*strDes)
		*strDes ++;  //将指针移动到最后
    
	while ((*strDes++ = *strSrc++) != '\0');

	return tmp;
}


void main()
{
    char src[10] = "world";
    char dest[100] = "hello ";

	char* str3 = strcat(dest, src);
	printf("%s", str3);  // hello world
}
```



#### 47.实现函数strncat

C库函数strncat可以实现把 **strSrc所指字符串的前n个字符**添加到strDes所指字符串结尾处,其函数定义;`char *strncat(char *strDes,const char *strSrc, Int count)`，请根据上述函数定义,实现此函数。

```c
char *strncat(char *strDes,const char *strSrc, int count)
{
	char *tmp = strDes; //赋值首地址，为了最后方便返回

	while (*strDes)
		*strDes ++;  //现将指针移动到最后

	// 把src所指字符串的前n个字符添加到dest结尾处(覆盖dest结尾处的'\0')
	while (count-- && (*strDes++ = *strSrc++));
	
    // 并添加'\0'
	*strDes = '\0';

	return tmp;
}

void main()
{
    char src[10] = "world";
    char dest[100] = "hello ";

	char* str3 = strncat(dest, src, 3);
	printf("%s", str3);
}
```



#### 48.实现函数strcpy

C库函数strcpy可以实现两字符串的复制,其函数定义:`char *strcpy (char *strDes, const char *strSrc)`.请根据上述函数定义,实现此函效。

```c
char *strcpy (char *strDes, const char *strSrc)
{
	
    char *p=NULL;
    if(strDes == NULL || strSrc == NULL)
    {
        return NULL;
	}

	p = strDes;

	while((*strDes++ = *strSrc++) != '\0');	

	return p;
}



void main()
{
    char src[10] = "world";
	char dest[100] = {0};

	char* str3 = strcpy(dest, src);
	printf("%s", str3);
}

```



#### 49.实现函数strncpy

C库函数strncpy可以实现把strsrc所指字符串的前n个字符复制到strDes所指字符串,其函数定义:`char *strncpy(char *strDes, const char *strSrc, int count)`，请根据上述函数定义,实现此函数。

```c
char *strncpy(char *strDes, const char *strSrc, int count)
{
    char *p=NULL;
    if(strDes == NULL || strSrc == NULL) //判断是否为空（作为一个程序员最基本的严谨性）
    {
        return NULL;
	}

	p = strDes;

	while(count--)
	{
		*strDes++ = *strSrc++;
	}

	//如果目标字符串不是以\0结尾，就自己添加一个
	if (*(strDes) != '\0')
		*(strDes + 1) = '\0';

	return p;

}


void main()
{
    char src[10] = "world";
	char dest[10] = {0};

	char * str3 = strncpy(dest, src, 4);
	printf("%s\n", str3);
}

```



#### 50.实现函数strcmp

C库函数strcmp可以实现两字符串的比较，其函数定义:`int strcmp(const char *s, const char *t)`,请根据上述函数定义,实现此函数。

```c
//低级版本
int strcmp(const char *s, const char *t)
{
	while (*s !='\0' || *t != '\0')
	{
		if(*s > *t) return 1;
		else if(*s < *t) return -1;

		*s++;
		*t++;
	}
	return 0;
}

//改良版本
int strcmp(const char *s, const char *t)
{
    if(*s == NULL || *t == NULL)
    {
        return NULL;
	}

	while(*s && *t && (*s == *t))
	{
		s++;
		t++;
	}
	return *s - *t;
}




void main()
{
    char src[10] = "world";
	char dest[10] = "worla";

	int flag = strcmp(dest, src);
	printf("%d\n", flag);
}

```



#### 51.实现函数strncmp

C库语数strncmp 可以实现s所指字符串与t所指字符串的最多前n个字符的比较,其函数定义: `int strncmp(const char *s, const char *t, int count)`，请根据上述函数定义,实现此函数,

```c
int strncmp(const char *s, const char *t, int count)
{
    if(*s == NULL || *t == NULL)
    {
        return NULL;
	}
	
	//还需要判断count是否超出了两个str的长度


	while (count-- && (*s == *t))
	{
		s++;
		t++;
	}
	return *s - *t;

}


void main()
{
    char src[10] = "world";
	char dest[10] = "wzrlt";

	int flag = strncmp(dest, src, 3);
	printf("%d\n", flag);
}
```



#### 52.利用梯形法计算定积分 

其中,f(x)=x^3+3x^2-x+2.

**思想：**为求出总面积，先将区间[a, b]分成n小块，每一块的图形近似于一个小梯形。用梯形面积代替该小块图形的面积，然后进行累计，就能够得到曲线图形的近似面积。

```cpp
???不会不会
    
#include <stdio.h>
#include <math.h>

float collect(float s,float t,int m,float (*p)(float x));
float fun1(float x);
float fun2(float x);
float fun3(float x);
float fun4(float x);

int main()
{
    int n,flag;
    float a,b,v=0.0;
    printf("Input the count range(from A to B)and the number of sections.\n");
    scanf("%f%f%d",&a,&b,&n);
    printf("Enter your choice：'1' for fun1,'2' for fun2,'3' for fun3,'4' for fun4==>");
    scanf("%d",&flag);
    if(flag==1)
        v=collect(a,b,n,fun1);
    else if(flag==2)
        v=collect(a,b,n,fun2);
    else if(flag==3)
        v=collect(a,b,n,fun3);
    else
        v=collect(a,b,n,fun4);
    printf("v=%f\n",v);
    return 0;
}

float collect(float s,float t,int n,float (*p)(float x))
{
    int i;
    float f,h,x,y1,y2,area;
    f=0.0;
    h=(t-s)/n;
    x=s;
    y1=(*p)(x);
    for(i=1;i<=n;i++)
    {
        x=x+h;
        y2=(*p)(x);
        area=(y1+y2)*h/2;
        y1=y2;
        f=f+area;
    }
    return (f);
}

float fun1(float x)
{
    float fx;
    fx=x*x-2.0*x+2.0;
    return(fx);
}

float fun2(float x)
{
    float fx;
    fx=x*x*x+3.0*x*x-x+2.0;
    return(fx);
}

float fun3 (float x)
{
    float fx;
    fx=x*sqrt(1+cos(2*x));
    return(fx);
}

float fun4(float x)
{
    float fx;
    fx=1/(1.0+x*x);
    return(fx);
}
```





#### 53.编程计算空间上任意两点之间的距离

```c
#include <math.h>

// 两个点，要定义两个结构体
struct Point
{
	float x;
	float y;
	float z;

};
float length_point(struct Point p1, struct Point p2)
{
	float x, y, z, len;
	x = fabs(p1.x - p2.x);
	y = fabs(p1.y - p2.y);
	z = fabs(p1.z - p2.z);

	len = sqrt(x*x + y*y + z*z);

	return len;
}

//测试函数
void length_point_test()
{
	struct Point p1, p2;
	printf("请输入point1的: x,y,z: ");
	scanf("%f, %f, %f", &p1.x, &p1.y, &p1.z);

	printf("请输入point2的: x,y,z: ");
	scanf("%f, %f, %f", &p2.x, &p2.y, &p2.z);

	printf("两点距离：%f ", length_point(p1, p2));

}
```







#### 54.日期运算问题

定义一个表示日期的结构体类型，再分别定义函数完成下列功能(两个日期由键盘输入):

计算某一天是对应年的第几天,

这一年一共多少天,

计算两个日期之间相隔的天数。

```c
typedef struct 
{
	int year, month, day;
}Date;

//1.计算某年的天数
int yearday(int year)
{
	int yday;
	if(year%=4==0 && year%100 || year == 0)
		yday = 365;
	else
		yday = 365;

	return yday;
}

//计算某年二月份天数
int monthday(int year)
{
	int mday;
	if(year%=4==0 && year%100 || year == 0)
		mday = 29;
	else
		mday = 28;

	return mday;
}

//2.计算某一天日期是对应年的第几天
int dayofyear(Date d)
{
	int total = 0;
	int months[13] ={0,31,28,30,31,30,31,31,30,31,30,31};
	months[2] = monthday(d.year); //判断二月份的天数
	for (int i =0; i< d.month; i++)
	{
		total += months[i];
	}
	total = total + d.day;

	return total;
}


//比较两个日期之间大小
int cmpdate(Date d, Date s)
{
	int result;
	if(d.year == s.year)
	{
		if(d.month == d.month)
		{
			if(d.day == d.day)
				result = 0;
			else
				result = d.day - s.day;
		}
		else
			result = d.month - s.month;
	}
	else
		result = d.year - s.year;

	return result;
}

// 3.计算两个日期之间相隔的天数
int interday(Date d, Date s)
{
	int result, te, ts, total;
	int year, start, end, day;
	result = cmpdate(d, s); //先比较大小
	if(result>0)  //如果第一个日期大
	{
		start = s.year;
		end = d.year;
		te = dayofyear(d);
		ts = dayofyear(s);
	}
	else if(result <0)
	{
		start = d.year;
		end = s.year;
		te = dayofyear(s);
		ts = dayofyear(d);
	
	}
	else
	{
		return 0;
	}

	if(start == end)   //如果是同一年
		return abs(te - ts);
	else               //如果不是同一年
	{
		total = 0;
		for (int i = start; i <= end; i++)
		{
			day = yearday(i);
			if(i==start)
				total = total + day -ts;

			else if(i==end)
				total = total + te;
			else
				total = total+day;
		}
	
	}

	return total;

}



void cal_date()
{
	Date d1, d2;
	int y, n;
	printf("输入日期: \n");
	scanf("%d%d%d", &d1.year, &d1.month, &d1.day);
	scanf("%d%d%d", &d2.year, &d2.month, &d2.day);
	y = yearday(d1.year);
	n = dayofyear(d1);
	printf("%d days  %d\n", d1.year, y);
	printf("%d-%d-%d is the % day", d1.year, d1.month, d1.day, n);

	n = interday(d1, d2);
	printf("%d-%d-%d 和 %d-%d-%d 相差  %d", d1.year, d1.month, d1.day,  d2.year, d2.month, d2.day);

	printf("%d 天\n", n);
}


```



#### 55.统计各字符个数

输入一行字符，分别统计出其中英文字母、空格、数字和其它字符的个数。

```c
void count_char()
{
	char c;
	int letter=0, space=0, other=0, num=0;

	printf("请输入字符串: ");
	while ((c=getchar()) !='\n')
	{
		if(c == ' ')
			space ++;
		else if((c <= 'Z' && c >='A') || (c <= 'z' && c >='a'))
			letter ++;
		else if(c <= '9' && c >='0')
			num ++;
		else
			other ++;
	}

	printf("空格:%d, 字母:%d, 数字:%d, 其他:%d", space, letter, num, other);

}
```





#### 56.统计文件的字符数、单词数以及总行数

统计包括:
每行的**字符数**和**单词数**
**文件**的**总字符数**、**总单词数**以及**总行数**

```c
//http://c.biancheng.net/cpp/html/2823.html
int *getCharNum(char *filename,int  *totalNum)
{
	FILE *fp; 
	char buffer[1003]; //缓冲区，存储读取到的每行的内容
	int bufferLen;   //缓冲区存储内容的长度
	int i;
	char c; ///读取到的字符
	int isLastBlank = 1; //上一个字符是否是空格
	int charNum = 0;  //当前行的字符数
	int wordNum = 0; //当前行的单词数


	if((fp=fopen(filename, "rb")) == NULL)
	{
		printf("打开文件错误!!!");
		return NULL;
	
	}

	//统计
	printf("line    words    chars\n");
	//m每次读取一行数据，保存到buffer，每行最多能有1000个字符
	while ((fgets(buffer, 1003, fp)) != NULL)
	{
		bufferLen = strlen(buffer);
		//遍历缓冲区的内容
		for (i = 0; i < bufferLen; i++)
		{
			c = buffer[i];  //c是当前位置的字符
			if(c == ' ' || c == '\t') //遇到空格
			{
				!isLastBlank && wordNum;   //如果上一个字符不是空格，那么单词数+1
				isLastBlank = 1;
			}else if(c != '\n' || c != '\r')  // 忽略换行 
			{
				charNum++; //如果挤不上换行符也不是空格，字符数+1
				isLastBlank = 0;
			}
		}
		!isLastBlank && wordNum++;  //如果最后一个字符不是空格，单词数+1
		isLastBlank = 1; //每次换行重置为1
		//一行结束，计算总单词数、总字符数、总行数
		totalNum[0]++;  //总行数
		totalNum[1] += charNum; //总字符数
		totalNum[2] += wordNum; //总单词数

		printf("%-7d%-7d%-7d\n", totalNum[0], wordNum, charNum);
		//置0，重新进入下一行
		charNum = 0;
		wordNum = 0;
	}
	
	return totalNum;
}

void count_all_test()
{
	char filename[20];
	int totalNum[3] = {0,0,0}; //totalNum[0] 总行数， totalNum[1] 总字符数, totalNum[2] 总单词数
	printf("输入文件名: ");
	scanf("%s", filename);

	if(getCharNum(filename, totalNum))
	{
		printf("%d行，%d个字符, %d个单词 \n", totalNum[0], totalNum[2], totalNum[1]);
	
	}else
	{
		printf("错误");
	}

}
```



#### 57.统计英文字母、空格、数字和其它字符的数目

输入一行字符,分别统计出其中英文字母、空格、数字和其它字符的个数。

```c
void count_char()
{
	char c;
	int letter=0, space=0, other=0, num=0;

	printf("请输入字符串: ");
	while ((c=getchar()) !='\n')
	{
		if(c == ' ')
			space ++;
		else if((c <= 'Z' && c >='A') || (c <= 'z' && c >='a'))
			letter ++;
		else if(c <= '9' && c >='0')
			num ++;
		else
			other ++;
	}

	printf("空格:%d, 字母:%d, 数字:%d, 其他:%d", space, letter, num, other);

}
```



#### 58.大小写转换并输入到文件保存

从键盘输入一个字符串,将小写字母全部转换成大写字母，大写字母全部转换成小写字母，然后输出到一个磁盘文件“test”中保存。输入的字符串以`!`结束。

```c
void save_char()
{
	FILE *fp;
	char str[100], filename[10];
	int ii;

	if((fp = fopen("D:\\2.txt", "w+")) == NULL)
	{
		printf("打开文件错误");
		exit(0);
	
	}

	printf("输入字符串: ");
	gets(str);

	while (str[ii] != '!')
	{
		if(str[ii] >= 'a' && str[ii] <='z')
			str[ii] = str[ii] + 'A'-'a';
		if(str[ii] >= 'A' && str[ii] <='Z')
		    str[ii] = str[ii] + 'a'-'A';

		fputc(str[ii], fp);
		ii++;
	}


	fclose(fp);


}


```



#### 59.利用递归方法求5!

f(n) = n*f(n-1)

```c
int get_5sum(int n)
{
	if(n==1)
		return 1;
	else
	{
		return n * get_5sum(n-1);
	}
}


void main()
{
	printf("5!= %d", get_5sum(5));
}

```



#### 60.输入三个字符串,将其按照从小到大排序并输出

```c
void swap(char *p1, char *p2)
{
	char p[20];  // 交换tmp变量
	strcpy(p, p1);
	strcpy(p1, p2);
	strcpy(p2, p);

}

void sort_3str()
{
	char s1[20], s2[20], s3[2];
	printf("输入第1个字符串: ");
	scanf("%s", s1);

	printf("输入第2个字符串: ");
	scanf("%s", s2);

	printf("输入第3个字符串: ");
	scanf("%s", s3);
	

	//开始比较
	if(strcmp(s1,s2) >0)  //如果S1>S2,则交换
		swap(s1, s2);
	if(strcmp(s1,s2) >0)   //如果s1>s3,则交换
		swap(s1, s3);
	if(strcmp(s2,s3) >0)   //如果s2>s3,则交换
		swap(s2, s3);

	printf("排序后的字符串: \n");
	printf("%s \n %s \n %s", s1, s2, s3);

}
```



























---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/c%E8%AF%AD%E8%A8%80%E5%81%9A%E9%A2%98%E8%AE%B0%E5%BD%95-%E5%85%A5%E9%97%A8%E7%89%88/  

