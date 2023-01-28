# C语言简单刷题2(未完)




#### 61.	求两个数的最大公约数和最小公倍数

```c
//1.求两个数的最大公约数和最小公倍数
int min_max(int x, int y)	
{
   //辗转相除法
	int tmp;

	/* 余数不为0，继续相除，直到余数为0 */
	while (x%y != 0)
	{
		tmp = y; //b在下一轮中作为除数，即是下一轮中的a，所以先闪到一边去
		y = x%y; //a,b的余数作为下一轮中的b,由a%b来取得
		x = tmp; //刚才tmp中存储了b的值，现在拿出来当做下一轮中的a使用

	}
	
	return y;
}

void min_max_test()
{
	int m,n, min;

	printf("使用辗转相除法求算\n");
    printf("请输入两个整数，如（7 24）：");
    scanf("%d%d", &m, &n);

	while (m <= 0 || m <= 0)
    {
        printf("输入有误，请重新输入：\n\t");
        scanf("%d%d", &m, &n);
    }


	min = min_max(m, n);

	printf("用辗转相除法求得 最大公约数为:%d\n", min);
    printf("\n\t\t 最小公倍数为:%d\n", m*n / min);

}
```



#### 62.在字符串种的所有数字字符前加一个$字符。

例如：输入`ARB23CD45`输`A$1B$2$3CD$4$5`

```c
#include <string.h>s

void add_daola(char *s)
{
	char tmp[80];
	int i, j;

	for (i = 0;s[i]; i++)  //复制s数组到t数组中
	{
		tmp[i] = s[i];
	}
	tmp[i] = '\0';

	//遍历
	for (i = 0, j=0; tmp[i]; i++)
	{	
		//如果遇到数字，前面先把$加入s，再加入数字
		if(tmp[i] >='0' && tmp[i] <= '9')
		{
			s[j++] = '$';
			s[j++] = tmp[i];	
		}
		//如果不是数字，则正常加入
		else
			s[j++] = tmp[i];
	}
	s[j++] = '\0';
}


void add_daola_test()
{
	char s[80];
	printf("请输入字符串：");
	scanf("%s", s);

	add_daola(s); //调用

	printf("处理结果：");
	puts(s);
}

```







#### 63.一个整数，它加上100后是个完全平方数，再加上168又是一个完全平方数，请问该数是多少?

```c
#include <math.h>

void full_sqart()
{
	long int i, x,y,z;
	for (i = 0; i < 10000; i++)
	{
		x = sqrt((double)(i+100));  // x为加上100然后开方后的结果
		y = sqrt((double)(i+168));

		if(x*x == i+100 && y*y == i+168) //如果一个数的平方根的平方等于该数，......
		{
			printf("%ld是完全平方数\n", i);
		}
	}
}
```





#### 64.质因数分解: 将一个正整数分解质因数。

如:输入90,打印出90=2\*3\*3\*5.

```c
//???没太懂
void zzhiyunshu()
{
	int i, n;

	printf("请输入正整数：");
	scanf("%d", &n);
	
	for (i=2; i <= n; i++)
	{
		while (n != i)
		{
			if( n % i == 0)
			{
				printf("%d*", i);
				n = n/i;   // ???
			}
			else
				break;
		}
	}
	printf("%d\n", n);
}

```





#### 65.自由落体求高度问题：

一球从 100米高度自由落下，每后次落地后反跳回原高度的一半再落下， 求它在第10次落地时，共经过多少米?第10次反弹多高?

```c
void drop_heigth()
{
	float sn = 100, hn = sn/2;
	for (int i = 0; i <=10; i++)
	{
		sn = sn + 2*hn;  //sn走过的距离(第i次落地时公经过的米数)
		hn = hn /2;      // 第n次反跳高度
	}
	printf("在第10次落地时，共经过多少米: %f \n", sn);
	printf("第10次反弹多高：%f \n", hn);
}

```







#### 66.求亲密数

如果整数A的全部因子(包括1，不包括A本身)之和等于B;且整数B的全部因子(包括1,不句括B本身)之和等于A,则将整数A和B称为亲密数。求3000以内的全部亲密数。

```c
// 对于这类多次将某些值存储到一个变量中时，一定要注意变量赋初值的位置。


void honey_num()
{
	int a, b, n, i;
	for (a = 1; a < 3000; a++)
	{
		for (b=0, i = 1; i < a/2; i++)
			if(!(a%i))
				b += i;   //将a的因子累加和存到b中

		for (n=0, i = 1; i < b/2; i++)
			if(!(b%i))
				n += i;   //将b的因子累加和存到n中

		//判断
		if(a == n && a<b)  //使得每对亲密数只输出一次
			printf("%d --- %d", a,b);  //如果n和a相等，则a和b是一对亲密数
	}
}

```



67. #### 使用递归逆序输出

利用递归函数调用方式，将所输入的5个字符，以相反顺序打印出来。

```c
// 方法1：
void revsese_num_out(int n)
{
	char ch;
	if(n <=1)
	{
		ch = getchar();
		putchar(ch);
	}
	else
	{
		ch = getchar();
		revsese_num_out(n-1);
		putchar(ch);
	}
}




void main()
{
	revsese_num_out(3);
}




//方法2：
void reverse_str(char *s, int n)
{
	int i;
	if((i=n/10) != 0)
		reverse_str(s+1, i);
	*s = n%10+'0';
}


void main()
{
	
    int num;

    char str[10]="";

    printf("input integer data：");

    scanf("%d",&num);

	reverse_str(str,num);

    printf("output string：\n");

    puts(str);
}

```





#### 68.猜年龄问题

有5个人坐在一起，问第五个人多少岁?他说比第4个人大2岁。问第4个人岁数，他说比第3个人大2岁。问第三个人，又说比第2人大两岁。问第2个人，说比第一个人大两岁。最后问第一个人，他说是10岁。请问第五个人多大?

```c
int guess_age(int n)
{
	if(n ==1)
		return 10;
	else
		return guess_age(n-1) + 2;
}


void main()
{
	printf("第5个人年龄: %d \n", guess_age(5));
}

```







#### 69.写一个递归程序求具有n个元素的整型数组R的和

```c
int get_sum(int R[], int n)
{
	if(n==1)
		return R[0];
	else
		return (R[n-1] + get_sum(R, n-1));
}

void get_sum_test()
{
	int R[5] = {11,22,33,44,55};
	printf("求和: %d", get_sum(R, 5));
}
```





#### 70.写一个递归程序求具有n个元素的整型数组R中最大值

```c
int get_max(int R[], int n)
{
	if(n==1)
		return R[0];
	else
	{
		if(R[n-1] >= get_max(R, n-1))
			return R[n-1];
		else
			return get_max(R, n-1);
	}

}

void get_max_test()
{
	int R[5] = {11,22,33,44,55};
	printf("最大值:%d \n", get_max(R, 5));

}

```





#### 71.写一个递归程序求具有n个元素的整型数组R的平均值

```c
int get_avg(int R[], int n)
{
	if( n==1)
		return R[0];
	else
		return(R[n-1]  + get_avg(R,n-1)*(n-1))/n;   //这里不太懂

}

void get_avg_test()
{
	int R[5] = {11,22,33,44,55};
	printf("平均值:%d \n", get_avg(R, 5));

}
```





#### 72.给一个不多于5位的正整数，要求:

一、 求它是几位数，

二、逆序打印出各位数字。

```c
void process_5num()
{
	int num, n=0;
	printf("输入一个不超过5位的正整数：");
	scanf("%d", &num);

	printf("逆序打印: ");
	while (num)
	{
		n++;
		printf("%d", num%10);
		num = num / 10;
	}
	printf("\n位数: %d", n);

}

// 4%10 = 4
```





#### 73.编写一个递归程序输入一个非负整数，返回返回组成它的数字之和

例如，调DigitSum(1729), 则应该1+7+2+9 它的和是19

```c
int DigitSum(int n)
{
	if(n==0)
		return 0;
	else
		return (n%10 + DigitSum(n/10));
}

void DigitSum_test()
{
	int num = 1729;
	printf("数字之和为：%d\n", DigitSum(num));
	
}

```





#### 74.有n个整数，使其前面各数顺序向后移动m个位置。最后m个数变成最前面的m个数

> 还是自己写不出来

```c
//123456
//m=2
// 56 1234
// 6-2+1=5

void move(int *x, int n, int m)
{
	int tmp[255];
	int i;
	// int *p:指针循环变量p
	for (i = 0; i < n; i++)
	{
		tmp[i] = x[i];  //先把arr的值复制给tmp数组一份
	}
	for (i = 0; i < m; i++)
	{
		x[i] = tmp[n-m+i];  // 最后m个数变成最前面的m个数  x[0]=tmp[6-2+1]
	}
	for (i = m; i <n; i++)
	{
		x[i] = tmp[i-m]; // 前面各数顺序向后移动m个位置   x[2]=tmp[2-2]
	}
}

void main()
{

	int a[20];
	int n,m, i;

	while (scanf("%d%d", &n, &m) != EOF)
	{
		for (i = 0; i < n; i++)
		{
			scanf("%d", &a[i]);

		}
		move(a, n, m);
		for (i = 0; i < n; i++)
		{
			printf("%d  ", a[i]);
			//printf("%d", a[n-i]);
			printf("\n");
		}
	}

}


```



#### 75.三天打鱼两天晒网问题

如果个渔夫从2019年1月1日开始每三天打一次渔，两天晒一次网，编程实现当输入2019月1日以后的任意一天，输出该渔夫是在打渔还是在晒网

```c
// 打渔，打渔，打渔，晒网，晒网
//自定义函数判断输入年份是否为闰年
int leap_year(int year)
{
	if(year%4==0 && year%100!=0 || year%400==0)
		return 1;
	else
		return 0;	
}

//自定义函数计算距离2019年1月1日共有多少天
int number(int year, int month, int day)
{
	int sum=0, i,j,k;
	int a[12] = {31,28,31,30,31,30,31,31,30,31,30,31}; //存放平年每月的天数
	int b[12] = {31,29,31,30,31,30,31,31,30,31,30,31}; //存放闰年每月的天数

	if(leap_year(year) ==1) //是否为闰年
		for (i = 0; i < month-1; i++)
		{
			sum += b[i];// 是闰年，则累加数组b的前m-1个月份的天数
		}

	else
        for (i = 0; i < month-1; i++)
        {
            sum += a[i];// 不是闰年，则累加数组a的前m-1个月份的天数
        }

	//
		for (j = 2019; j < year; j++)
		{
			if(leap_year(j) == 1)
				sum += 366; //2019年到输入的年份中，是闰年的加366
			else
				sum += 365; // 不是闰年的加365
		}

		sum += day; //前面累加的结果加上日期，求出总天数

		return day;
}

void fish_main()
{
	//测试函数
	int year, month, day, n;
	printf("请输入日期: ");
	scanf("%d%d%d", &year,&month, &day);

	n = number(year, month, day); //调用
	if((n%5)<4 && (n%5)>0) //余数是1或2或3的时候，说明在打渔，否则在晒网
		printf("%d-%d-%d 在打渔", year, month, day);
	else
		printf("%d-%d-%d 在晒网", year, month, day);
}

```



#### 76.八进制转换为十进制

```c
// 例如，将八进制数字 53627 转换成十进制：
// 53627 = 5×8^4 + 3×8^3 + 6×8^2 + 2×8^1 + 7×8^0 = 22423（十进制）

// F1.
void o_to_ten()
{
	char *p, s[6];
	int n=0, sum=0; //10进制
	int i=0;
	
	p = s;
	gets(p); // 8进制  53627 

	while (*(p)!='\0')
	{
		n = n * 8 + *p - '0';
		p++;
	}

	printf("%d",n);


}


//F1:或者下面这种笨方法
void main()
{
	// a是八进制数字
	int a = 53627, count=0, sum=0; 
	int arr[6] = {};
	while (a)
	{
		printf("%d\n", a%10);
		arr[count] = (a%10)*(int)pow((double)8, count);
		++count;
		a = a /10;
	}

	printf("个数: %d\n", count);

	for (int j = 0; j < 5; j++)
	{
		printf("%d ", arr[j]);
		sum += arr[j];
	}
	printf("十进制: %d\n", sum);

}

```



#### 77.有两个磁盘文件A和B,各存放一行字母，要求把这两个文件中的信息合并(按字母顺序排列），输出到一个新文件C中。

解答：将A、B文件中数据读书放到数组中，再对数组进行排序，并输出到文件中

```c
void merge_file()
{
	FILE *fp;
	int i,j, n, ni;
	char c[160], t, ch; //用c数组保存合并结果，再把合并结果放到文件c中

	//读取A文件内容
	if((fp=fopen("D:\\A.txt", "r")) == NULL)
	{
		printf("file A error");
	}

	printf("\n A文件内容为: \n");
	for(i=0; (ch=fgetc(fp))!=EOF; i++)
	{
		c[i] = ch;
		putchar(c[i]);
	}
	fclose(fp);

	//读取B文件内容
	if((fp=fopen("D:\\B.txt", "r")) == NULL)
	{
		printf("file A error");
	}

	printf("\n B文件内容为: \n");
	for(i; (ch=fgetc(fp))!=EOF; i++)
	{
		c[i] = ch;
		putchar(c[i]);
	}
	fclose(fp);


	//排序
	n = i;
	for(i=0; i<n; i++)
	{
		for(j=i+1; j<n; j++)
		{
			if(c[i]>c[j])
			{
				t = c[i];
				c[i] = c[j];
				c[j] = t;
			}
		}
	}
	//
	printf("\n C文件内容:\n");
	fp = fopen("d:\\C.txt", "w");
	for(i=0; i<n; i++)
	{
		putc(c[i], fp);
		putchar(c[i]);
	}
	fclose(fp);


}

```





#### 78.结构体数据的文件写入问题

有五个学生，每个学生有3门课的成绩，从键盘输入以上数据（包括学生县,姓名，二门课成绩)计算出平均成绩，况原有的数据和计算出的平均分数存放在磁盘文件中。

```c
struct student
{
	char num[10];
	char name[8];
	int score[3];
	float avg;
}stu[5];


void test_file()
{
	int i,j, sum=0;
	FILE *fp;
	for (i = 0; i < 5; i++)
	{
		printf("--->请输出第%d个学生的成绩: \n", i+1);
		printf("学号:");
		scanf("%s", stu[i].num);

		printf("姓名:");
		scanf("%s", stu[i].name);

		//三科的成绩的录入
		printf("-->录入成绩: \n");
		for(j=0; j<3; j++)
		{
			printf("输入第%d科成绩:", j+1);
			scanf("%d", &stu[i].score[j]);
			sum += stu[i].score[j];

		}
		stu[i].avg  = sum/3.0;
	}

	//成绩写入文件中
	if((fp=fopen("D:\\score.txt","w"))==NULL)
	{
		printf("can't open file\n");
		exit(0);
	}
	for (int i = 0; i < 5; i++)
	{
	    if(fwrite(&stu[i], sizeof(struct student), 1, fp) != 1)
			printf("write file error\n");
	}
	fclose(fp);


	//读取文件内容到结构体数据块中
	printf("====================================\n");
	fp=fopen("D:\\score.txt","r");
	for (i = 0; i < 5; i++)
	{
	    fread(&stu[i], sizeof(struct student), 1, fp);
		printf("学号:%s, 姓名:%s, 分数1:%d, 分数2:%d, 分数3:%d, 平均分:%lf", stu[i].num, stu[i].name, stu[i].score[0], stu[i].score[1], stu[i].score[2], stu[i].avg);
	}
	fclose(fp);

}
```







#### 79.验证歌德巴赫猜想

**问题为:**2000以内的不小于4的正偶数都能够分解为两个素数之和(即验证歌德巴赫猜想对20000以内的正偶数成立)

**分析:**根据问题描述，为了验证歌德巴赫猜想对2000以内的正偶数都是成立的，要将整数分解为两部分，然后判断分解出的两个整数是否均为素数。若是，则满足题意，否则应重新进行分解和判断。

```c
//先判断是否为素数
int is_prime(int n)
{
    int i;
    if(n==2)
        return 1;  /*n是2，返回1*/
    if(n%2==0)
        return 0;  /*n是偶数，不是素数，返回0*/
	for(i=3; i<=(int)(double)sqrt((double)n); i+=2)
        if(n%i==0)
            return 0;  /*n是奇数，不是素数，返回0*/
    return 1;  /*n是除2以外的素数返回1*/
}

void gede()
{
	int n, ok, i;

	int t[102];
	t[0] = 6;
	int j = 1;
	for (int k = 0; k <=99; ++k)
	{
		t[k + 1] = t[k] + 1;  //只是一个赋初值的过程
	}	

	while (j<=99){
	//while (scanf("%d", &n) != EOF)
	//{
		++j;
		n=t[j];
		ok = 0;  //先置标志位为0
		for (i = 2; i<=n/2; i++)
		{
			if(is_prime(n) == 1 && is_prime(n-i) == 1)  //相加之和为这个数的另一个数如果也是素数
			{
				printf("数字%d可由以下两个素数相加而来：\n",n);
				printf("%d %d\n", i, n-i); // n和n-i都是素数，则打印
				ok = 1;
			}
			if(i != 2)
				i++;
			if(ok)
				break; //已打印出所需要的输出结果，跳出循环

		}
	//}
	}
}
```



#### 80.用1,2,3,4共四个数字，能组成多少个互不相同且无重复数字的三位数？

```c
void three_num()
{
	int count=0;

	for (int i = 1; i < 5; i++)
	{
		for (int j = 1; j < 5; j++)
		{
			for (int k = 1; k < 5; k++)
			{
				if(i!=j && i!=k && j!=k)
				{
					count += 1;
					printf("%d%d%d ", i,j,k);

					//美观，每10个换一次行
					if(count%10 == 0)
						printf("\n");
				}
			}
		}
	}
	printf("共有%d组", count);
}
```





#### 81.约瑟夫环问题

编号为1，2，3，… n的n个人围坐一圈，任选一个正整数 m 作为报数上限值，从第一个人开始按顺时针方向报数，报数到 m 时停止，报数为m的人出列。从出列人的顺时针方向的下一个人开始又从1重新报数,如此下去,直到所有人都全部出列为止。
**解答:**函数中利用循环访问数组中n个元素，每次访问元素，设定内循环连续访问m 个元素,元素访问的下标为k,访问到第m个元素时，如果元素不是0,此时输出元素 a[k],再设定 a[K]为0，继续访问后面的元素。

```c
//???没懂

#define N 10
int ysfh(int a[], int n, int m)
{
	int i, j, k=0;
	for (i = 0; i < n; i++)
	{
		j = 1;
		while (j<m) //报1,2...m-1的人
		{
			while (a[k]==0)
			{
				k = (k+1)%n;
			}
			j++;
			k = (k+1)%n;  //类比循环队列，对n求余能够实现循环
		}
		while (a[k] == 0)
		{
			k = (k+1)%n; 
		}
		printf("%d ", a[k]);  //输出依次被淘汰的人
		a[k]=0;
	}
	
	return 0;
}


void ysfh_test()
{
	int arr[100];
	int i,j,m,n;
	printf("输入n和m: ");
	scanf("%d%d", &n,&m);
	for (i = 0; i < n; i++)  
	{
		arr[i] = i+1;   //生成初始化数组
	}
	ysfh(arr, n, m);   //调用

	printf("\n");
}
```



#### 82.狼抓兔子问题

一只兔子躲进了10个环形分布的洞的某一个,狼在第一个洞没有找到免子,就隔一个洞,到第个洞去找,也没有找到，就隔两个洞，到第六个洞去找,以后每次多隔一个洞去找兔子……这样下去,结果一直找不到兔子,请问:兔子可能躲在哪个洞中?（设最大寻找次数为1000）



```c
void walf()
{
	
	int a[10];                     //10个洞 
	int i,n=0,x=0;                 //n->累加洞编号 x->洞下标 

	for(i=0;i<10;i++)              //数组初始化为1，标记兔子可能藏在各个洞 
		a[i]=1;

	for(i=0;i<1000;i++)            //设最大寻找次数为1000 
	{
		n+=(i+1);                  //每次多隔1个洞 
		x=n%10;                    //取余，往回找 
		a[x]=0;                    //访问过的洞标记为0，说明没有兔子 
	}

	for(i=0;i<10;i++)             
	{
		if(a[i])                   //狼未访问过的洞，说明可能有兔子 
			printf("兔子可能在第%d个洞中\n",i+1);

	}


}

```



#### 83.马克思手稿中的数学题

马克思手稿中有一道趣味数学问题：有30个人，其中有男人、女人和小孩，他们在同一家饭馆吃饭，总共花了50先令。已知每个男人吃饭需要花3先令，每个女人吃饭需要花2先令，每个小孩吃饭需要花1先令，请编程求出男人、女人和小孩各有几人。

**分析：**

![image-20210418074530569](C:\Users\19401\AppData\Roaming\Typora\typora-user-images\image-20210418074530569.png)



在问题分析中我们抽象出了一个不定方程组，显然得到了不定方程组的解，该问题也就解决了。但不定方程组中包含了 x、y、z 这3个变量，而方程只有两个，因此不能直接求出x、y、z的值。

而由方程 (3)，我们得到了x的取值范围，因此可将x的有效取值依次代入不定方程组中（即方程 (1)、(2) 和 (3)）中，能使3个方程同时成立的解即为该问题的解。为实现该功能,只需使用一个for循环语句即可。



**代码：**

```c
void mxs_math()
{
	int x, y, z, number=0;

	printf("男人  女人  孩子");

	for (x = 0; x <= 10; x++)   //已经x的范围，则就在这个范围内依次遍历取有效值
	{
		y = 20 - 2*x;   //表示y
		z = 30 - x - y;  //表示z

		if(3*x + 2*y + z == 50)  //如果当前x, y, z的取值满足30个人吃饭总共花了50先令，则输出
			printf("%2d:%4d%5d%6d\n", ++number, x, y, z);
	}
}

```



#### 84.爱因斯坦台阶问题

爱因斯坦曾经提出过这样一道有趣的数学题：

有一个长阶梯，若每步上2阶，最后剩下1阶（即x%2 = 1）；

若每步上3阶，最后剩2阶（x%3 = 2）；

若每步上5阶，最后剩下4阶（x%5 = 4）；

若每步上6阶，最后剩5阶（x%6 = 5）；

只有每步上7阶，最后刚好一阶也不剩（x%7 = 0）。

请问该阶梯至少有多少阶？

```c
void aiyinsitan()
{
	int i; // 阶梯数
	int count =0;   //记录满足条件的阶梯个数
	for (i = 1; i < 1000; i++)
	{
		if(i%2==1 && i%3==2 && i%4==3 && i%5==4 && i%6==5 && i%7==0)
		{
			count ++;
			printf("%d 可以\n", i);
		}
	}
	printf("共有%d个台阶满足条件\n", count);
}

```





#### 85.牛顿迭代法求方程根

>谭浩强C程序设计（第四版）p218第12题

方程ax^3+bx^2+cx+d=0，系数a，b，c，d由主函数输入。求x在1附近的一个实根，由主函数输出。

牛顿的迭代法公式是：x = x0-f(x0)/f’(x0), 设迭代到`|x - x0| <= 10^-5`结束

```c
float niudun(float a, float b, float c, float d)  //系数由外部手动传入
{
	float x0, x=1.5, f, fd, h; // f为方程的值，fd为方程求导后的值

	do
	{
		x0 = x; //用所求的的x的值来代替x0原来的值
		f = a*x0*x0*x0 + b*x0*x0 + c*x0 + d;
		fd = a*x0*x0 + b*x0 + c;
		h = f / fd;
		x = x0 - h;
	}while (fabs(x-x0) <= 1e-5);
	
	return x;
}

```



#### 86.枚举排列组合

给定集合`S={a,b,c,d,e,f,g}`, 请编写一段程序枚举出其中任意三元素构成的所有排列组合，如acd, dea等。

**分析:**枚举法一般需要设置多个循环，需要枚举几个元素就嵌套几层循环，在各层循环之间设置题目要求的限定关系输出即可(比如本题的各元素不能相同等)

```c
void sorAndCombination(char str[], int n)
{
	int count = 0;
	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < n; j++)
		{
			for (int k = 0; k < n; k++)
			{
				if ((str[i] != str[j]) &&  (str[i] != str[k]) && (str[j] != str[k]))
					printf("%c%c%c\n", str[i], str[j], str[k]);
				    count ++;
			}
		}
	}

	printf("共%d种组合", count);
}

void sorAndCombination_test()
{
	char s[] = "abcd";
	sorAndCombination(s, 4);
}
```



#### 87.乒乓球对手匹配问题

两个乒乓球队进行比赛，各出三人。甲队为a,b,c三人，乙队为x,y,z三人。已抽签决定
比赛名单。有人向队员打听比赛的名单。a说他不和x比，c说他不和x,z比，请编程序找出三队赛手的名单。

**解答:**通过循环嵌套校举出可能的对战顺序，再用if语句判断避免参赛的队员重复，最后通过题目要求筛选出符合题意的对手。

```c
void ball()
{
	for (char i = 'X'; i <= 'Z'; i++)
	{
		for (char j = 'X'; j <= 'Z'; j++)
		{
			for (char k = 'X'; k <= 'Z'; k++)
			{
				if ((i != j) &&  (i != k) && (j != k))  //避免队员重复
				{
					if((i!='X') && (k!='X') && (k!='Z'))
					{
						printf("A --> %c\n", i);
						printf("B --> %c\n", j);
						printf("B --> %c\n", k);
					}
				}
			}
		}
	}
}

```



#### 88.换分币问题

将5元的人民币兑换成1元、5角和1角的硬币。共有多少种不同的兑换方法

```c
void money()
{
	//int sum = 50, count=0;  // //将单位全部转化成角
	int count = 0;
	for (int i = 0; i <=50; i+=10)  //i为为1元硬币钱数，其取值为0,10,20,30,40,50(以角为单位)
	{
		for (int j = 0; j <=50; j+=5)   // j为5角硬币钱数，其取值为0,5,10,15,20,25,30,35,40,,45,50
		{
			for (int k = 0; k <=50; k++)   // k为1角硬币钱数，其取值为0,1,...50
			{
				if (i+j+k == 50)
				{
				    //printf("10*%d+5*%d+1*%d\n", i/10, j/5, k);		
					printf("1元*%d + 5角*%d + 1角*%d\n", i/10, j/5, k);	
				    count ++;
				}			 
			}
		}
	}

	printf("共%d种兑换方法", count);
}
```



#### 89.百钱买百鸡问题

中国古代数学家张丘建在他的(算经)中提出了一个著名的“百钱买百鸡问题”，鸡翁一，值钱五；

鸡母一，值钱三；鸡雏三，值钱一。百钱买百鸡，问翁、母、雏各几何?

```c
void chicken()
{
	// 可以得到方程组如下：
	/*
	5x + 3y + 1/3*z = 100
	x + y + z =100
	0 <= x <= 100
	0 <= y <= 100
	0 <= z <= 100
	*/

	//if( (x*5 + y*3 + z/3) == 100  && z%3==0  && x+xy+z==100)
	for (int i = 0; i <= 100; i++)
	{
		for (int j = 0; j <= 100; j++)
		{
			for (int k = 0; k <= 100; k++)
			{
				if((5*i + 3*j +k/3)==100  && k%3==0  && i+j+k==100)
				{
					
					printf("公鸡:%d, 母鸡:%d, 小鸡:%d\n", i, j, k);
				}
			}
		}
	}
}
```





#### 90.实现如下表所示的5-魔方阵

| 17   | 24   | 1    | 8    | 15   |
| ---- | ---- | ---- | ---- | ---- |
| 23   | 5    | 7    | 14   | 16   |
| 4    | 6    | 13   | 20   | 22   |
| 10   | 12   | 19   | 21   | 3    |
| 11   | 18   | 25   | 2    | 9    |

**问题分析:**

所谓“n-魔方阵”，指的是使用1〜n2共n2个自然数排列成一个n×n的方阵，其中n为奇数；该方阵的每行、每列及对角线元素之和都相等，并为一个只与n有关的常数，该常数为n×(n2+1)/2。



```c
// ???
/*
例如4-魔方阵，其第一行、第一列及主对角线上各元素之和如下：

第一行元素之和：17+24+1+8+15=65

第一列元素之和：17+23+4+10+11=65

主对角线上元素之和：17+5+13+21+9=65

而 n×(n2+1)/2=5×(52+1)/2=65 可以验证，5-魔方阵中其余各行、各列及副对角线上的元素之和也都为65。

假定阵列的行列下标都从0开始，则魔方阵的生成方法为：在第0行中间置1，对从2开始的其余n2-1个数依次按下列规则存放：

(1) 假定当前数的下标为(i，j)，则下一个数的放置位置为当前位置的右上方，即下标为(i-1，j+1)的位置。

(2) 如果当前数在第0行，即i-1小于0，则将下一个数放在最后一行的下一列上，即下标为(n-1，j+1)的位置。

(3) 如果当前数在最后一列上，即j+1大于n-1，则将下一个数放在上一行的第一列上，即下标为(i-1，0)的位置。

(4) 如果当前数是n的倍数，则将下一个数直接放在当前位置的正下方，即下标为(i+1，j)的位置

*/


/*
【算法设计】
在设计算法时釆用了下面一些方法：

定义array()函数，array()函数的根据输入的n值，生成并显示一个魔方阵，当发现n不是奇数时，就加1使之成为奇数。

使用动态内存分配与释放函数malloc()与free()，在程序执行过程中动态分配与释放内存，这样做的好处是使代码具有通用性，同时提高内存的使用率。

在分配内存时还要注意，由于一个整型数要占用两个内存，因此，如果魔方阵中要存放的数有max个，则分配内存时要分配2*max个单元，从而有malloc(max+max)。在malloc()函数中使用max+max而不是2*max是考虑了程序运行的性能。显然应该使用二维数组来表示魔方阵，但虽然数组是二维形式的，而由于内存是一维线性的，因此在存取数组元素时，要将双下标转换为单个索引编号。在程序中直接定义了指针变量来指向数组空间，即使用malloc()函数分配的内存。

*/

//90.实现如下表所示的5-魔方阵
int mofang(int n)
{
	int i, j, no, num, max;
	int *mtrx;
	if(n%2 == 0) /*如果n是偶数，则加1使其变为奇数*/
		n=n+1;

	max=n*n;
	mtrx = (int *)malloc(max+max);  //为魔方阵分配内存
	mtrx[n/2]=1; /* 将1存入数组*/
	i=0; /*自然数1所在行*/
	j=n/2; /*自然数1所在列*/

	//从2开始确定每个数的存放位置
	for(num=2; num<=max; num++)
	{
		if((num-1)%n == 0) /*如果当前数是n的倍数 ？？？ */
		{
			i=i+2;
			j=j-1;
		}

		if(i<0) /*当前数在第0行*/
		{
			i=n-1;
		}

		if(j>n-1) /*当前数在最后一列，即n-1列*/
		{
			j=0;
		}

		no=i*n+j; /*找到当前数在数组中的存放位置*/
		mtrx[no]=num;
	}



	/*打印生成的魔方阵*/
	printf("生成的%d-魔方阵为:",n);
	no=0;
	for(i=0; i<n; i++)
	{
		printf("n");

		for(j=0; j<n; j++)
		{
			printf("%3d", mtrx[no]);
			no++;
		}
	}

	printf("n");
	free(mtrx);

	return 0;
}

void mofang_test()
{
	int n;

	printf("请输入n值:");
	scanf("%d", &n);

	mofang(n); /*调用函数*/

}






```







#### 91.给定一个有序数组，你要在原地删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度

![image-20210420234453370](C:\Users\19401\AppData\Roaming\Typora\typora-user-images\image-20210420234453370.png)

```c
// ???

int remove2(int* nums, int numsSize)
{
	int i;
	int j=1;
	int len=numsSize;
	int temp=nums[0];  // i用于遍历数组，j用于记录不同的元素到数组中，k用于记录不同的元素于数组中, temp基准值

	for (i = 1; i < numsSize; i++) //遍历数组
	{
		if(nums[i] != temp) //若元素不等于基值，将其放入数组前方，并且作为新基准值
		{
		   nums[j++]=nums[i];
		   temp = nums[i];
		}
		else
			len--;  
	}

	return len;
}



void main()
{
	int nums[11] = {1,1,2,3,3,4,5,6,6,6,7};  //给定一个有序数组
	printf("\n新长度:%d", remove2(nums, 11));

}
```

```c
//添加个打印新的的元素的功能
void remove2(int* nums, int numsSize)
{
	int i;
	int j=1;
	int len=numsSize;
	int temp=nums[0];  // i用于遍历数组，j用于记录不同的元素到数组中，k用于记录不同的元素于数组中, temp基准值

	for (i = 1; i < numsSize; i++) //遍历数组
	{
		if(nums[i] != temp) //若元素不等于基值，将其放入数组前方，并且作为新基准值
		{
		   nums[j++]=nums[i]; // 是先把nums[j]=nums[i], 然后j再加1是吗
		   temp = nums[i];
		}
		else
			len--;  
	}
	printf("新长度为: %d\n", len);

	//打印元素
	printf("新元素为: ");
	for(i=0; i<len; i++){
        printf("%d",nums[i]);
    }
	printf("\n");
}

void main()
{
	int nums[11] = {1,1,2,3,3,4,5,6,6,6,7};  //给定一个有序数组
	remove2(nums, 11);
}
```







#### 92.三旗问题

假设有一条绳子，上面有红、白、蓝三种颜色的旗子，起初绳子上的旗子的颜色并没有顺序，你希望将之分类并排列为**蓝、白、红的顺序**，要如何移动次数最少，注意你只能在绳子上进行这个动作，并且一次只能调换两个旗子。

**翻译成白话就是这个意思：**有123三个数，每个数字的个数不定，数字间顺序不定，要求用最少的步数将他改成111222333这种情况，即相同数字挨着，不同数字按顺序排列。



```c
/*
如果排列顺序是b，w，r。我们如下定义这三个指针的意义：b永远指向第一个不是b的元素（即从开始位置起，第一个不是b的元素），w作为遍历的游标，往下运行，并判断该元素是不是w，如果是不做处理，继续往下运行，如果是b则与b指向的元素交换，如果是r则与r指向的元素交换。
r永远指向第一个不是r的元素，不同的是，r是从这个数列的最后开始的，因为我们最终要将r放到后面。
如此一次遍历该数列时，
遇到第一个元素如果是b，则w和b继续往下走，
如果是w，则b指向该元素，w往下走，
如果是r，则在r的地方找到第一个不是r的元素并交换，
以此类推，遇到w继续走，遇到b往前移，遇到r往后移

*/

#define BLUE 'b'
#define WHITE 'w'
#define RED 'r'


#define SWAP(x,y){char temp; temp = color[x]; color[x] = color[y]; color[y] = temp;}




```





#### 93.猴子吃桃问题

猴子第一天摘下若干个桃子，当即吃了一半，还不过瘾，又多吃了一个。**第二天早上又将第一天剩下的桃子吃掉一半，又多吃了一个**。以后每天早上都吃了前一天剩下的一半零一个。到第 10 天早上想再吃时，发现只剩下一个桃子了。编写程序求猴子第一天摘了多少个桃子。

```c
// 递归方式
int monkey(int day) //返回第day天时的桃子数
{
	int num;
	if(day == 10)
		return 1;
	else
	{
		return 2*(monkey(day+1) + 1);
		
	}
}

void main()
{
    printf("第1天的桃子数：%d\n", monkey(1));
    
	//for(int i=1;i<=10;i++)
	//{
	//	printf("第%d天的桃子数：%d\n", i, monkey(i));
	//
	//}
}
```

```c
//循环方式
#include<stdio.h>

int main(void)
{
    int day, x1, x2=1; //第10天1个桃子
    for(day=10;day>=1;day--)
    {
        printf("第%d天的桃子数为：%d ",day,x2);
        x1=2*(x2+1);
        x2=x1;
    }
}
```





#### 94.求黑洞数

编程求三位数中的“黑洞数”。

黑洞数又称陷阱数，任何一个数字不全相同的整数，经有限次“重排求差”操作，总会得到某一个或一些数，这些数即为黑洞数。“重排求差”操作是将组成一个数的各位数字重排得到的**最大数**减去**最小数**，例如207，“重排求差”操作序列是720-027=693，963-369=594，954-459=495，再做下去就不变了，再用208算一次，也停止到495，所以495是三位黑洞数。

**问题分析:** 根据“黑洞数”定义，对于任一个数字不全相同的整数，最后结果总会掉入到一个黑洞圈或黑洞数里，最后结果一旦为黑洞数，无论再重复进行多少次的“重排求差”操作，则结果都是一样的，可把结果相等作为判断“黑洞数”的依据。

**算法设计：**

(1) 将任一个三位数进行拆分。
(2) 拆分后的数据重新组合，将可以组合的最大值减去最小值，差赋给变量j。
(3) 将当前差值暂存到另一变量h中：h=j。
(4) 对变量j执行拆分、重组、求差操作，差值仍然存储到变量j中。
(5) 判断当前差值j是否与前一次的差相等，若相等将差值输出并结束循环，否则，重复步骤 (3)、(4) 和 (5)。

```c
//求三个数中的最大值
int max_3(int a,int b,int c)
{
	int tmp;
	if(a<b)
	{ tmp=a; a=b; b=tmp; }
	if(a<c)
	{ tmp=a; a=c; c=tmp; }
	if(b<c)
	{ tmp=b; b=c; c=tmp; }

	return (a*100 + b*10 + c);

}

//求三个数中的最小值
int min_3(int a,int b,int c)
{
	int tmp;
	if(a<b)
	{ tmp=a; a=b; b=tmp; }
	if(a<c)
	{ tmp=a; a=c; c=tmp; }
	if(b<c)
	{ tmp=b; b=c; c=tmp; }

	return (c*100 + b*10 + a);

}

//测试函数
void heidong()
{
	int num, ge, shi, bai, max, min, j, k, h;
	printf("请输入一个三位数：");
	scanf("%d", &num);
    bai=num/100;
    shi=num%100/10;
    ge=num%10;

	max = max_3(bai, shi, ge);
    min = min_3(bai, shi, ge);
	j = max - min;

    for(k=0; ; k++)  /*k控制循环次数*/
	{
	    h=j;  /*h记录上一次最大值与最小值的差*/

		bai=num/100;
		shi=num%100/10;
		ge=num%10;
		max = max_3(bai, shi, ge);
		min = min_3(bai, shi, ge);
		j = max - min;

		if(j==h)  /*最后两次差相等时，差即为所求黑洞数*/
        {
            printf("%d\n", j);
            break;  /*跳出循环*/
        }
	}



}

```





#### 95.分鱼问题_1

**问题分析：**

假设5个人合伙捕了x条鱼，则“A第一个醒来，他将鱼平分为5份，把多余的一条扔回河中，然后拿着自己的一份回家去了”之后，还剩下4(x-1)/5条鱼。

这里实际包含了一个隐含条件：假设Xn为第n(n=1、2、3、4、5)个人分鱼前鱼的总数，则(Xn-1)/5必须为正整数，否则不合题意。（Xn-1)/5为正整数即(X〜l)mod5=0必须成立。

又根据题意，应该有下面等式：
X4=4(X5-1)/5
X3=4(X4-1)/5
X2-4(X3-1)/5
X1=4(X2-1)/5

则一旦给定X5，就可以依次推算出X4、X3、X2和X1的值。要保证X5、X4、X3、X2和X1都满足条件(Xn-1)mod5=0，此时的**X5**则为5个人合伙捕到的鱼的**总条数**。显然，5个人合伙可能捕到的鱼的条数并不唯一，但题目中强调了 “至少”合伙捕到的鱼，此时题目的答案唯一。该问题可使用递归的方法求解。

**程序设计:**

在main()函数中构建一个不定次数的do-while循环。定义变量x表示5个人合伙可能捕到的鱼的条数，可以取x的最小值为6，让x值逐渐增加，x每一次取值，都增加5，直到找到一个符合问题要求的答案。由于题目中问“这5人至少合伙捕到多少条鱼”，而我找到的第一个x值就是5个人至少捕到的鱼的总条数。

通过这个循环，就可以对每一个的可能情况进行检查。当然，是通过调用分鱼的递归函数来进行检查的。

分鱼的递归函数如下：
fish()函数中包含了两个参数：n和x。n表示参与分鱼的人数，x表示n个人分鱼前鱼的总条数。这两个参数都是由main()函数中传递进来的。

根据前面的分析，当n=5时，(x-1)mod5==0必须成立，否则该x值不是满足题意的值，退出fish()函数，返回到main()函数，main()函数中再传递新的x值到fish中进行检验。如果(x-1)mod5==0条件成立，则要判断n=4时，(x-1)mod5=0条件是否成立，需要注意的是，此时的形参x是4个人分鱼前鱼的总条数，即f(5,x)递归调用f(4,(x-1)/5*4)。这样依次进行下去，直到n=1时，(x-1)mod5==0条件仍成立，则说明开始从main()函数中传递进来的x值是符合题意要求的一个值，可以逐层从递归函数中返回，每次返回值都为1，直至返回到main()函数。

```c
int fish(int n, int x)
{
	if((x-1)%5 == 0)
	{
		if(n==1)
			return 1;
		else
			return fish(n-1, (x-1)/5*4);	
	}
	return 0;
}


void fish_test()
{
	int i=0, flag=0, x;
	do
	{
		 i += 1;
	     x=i*5+1;  /*x最小值为6，以后每次增加5*/
		 if(fish(5,x))  /*将x传入分鱼递归函数进行检验*/
		 {
		    flag=1;  /*找到第一个符合题意的x, 则置标志位为1*/
            printf("五个人合伙捕到的鱼总数为%d\n", x);
		 }
	
	}while (!flag);

}
```





#### 96.分鱼问题_2

甲、乙、丙三位渔夫出海打鱼，他们随船带了21只箩筐。当晚返航时，他们发现有7筐装满了鱼，还有7筐装了半筐鱼，另外7筐则是空的，由于他们没有秤，只好通过目测认为7个满筐鱼的重量是相等的，7个半筐鱼的重量是相等的。在不将鱼倒出来的前提下,怎样将鱼和筐平分为3份？



```c
//未完待续...

```











---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/c%E8%AF%AD%E8%A8%80%E7%AE%80%E5%8D%95%E5%88%B7%E9%A2%982-%E6%9C%AA%E5%AE%8C/  

