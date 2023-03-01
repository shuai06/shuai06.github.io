# 递归求具有n个元素的整型数组R的平均值




#### 递归求具有n个元素的整型数组R的平均值

今天做题的时候遇到这个不会的题，虽然题很简单单还是太菜了，记录下来，这还是问了一个小学妹之后才懂了，自卑ing... ...



```c
int get_agv(int R[], int n)
{
	if(n==1)
		return R[0];
	else
		return (R[n-1] + get_agv(R, n-1)*(n-1))/n;	   // n-1  用来约分母(约前面一个数的分母)， 把分母写在横线的下面比较容易看出来
}


void main()
{
	int R[5] = {22,44,66,88,10};
	printf("平均值: %d\n", get_agv(R, 5));
}

```



最后一句：return (R[n-1] + get_agv(R, n-1)*<font color=red>(n-1</font>))/n;中的n-1是用来约分母的(约前面一个数的分母)，比如下面这样更直观一点

![示意图](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E6%B1%82%E5%B9%B3%E5%9D%87%E5%80%BC%E9%A2%98.jpg)





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E9%80%92%E5%BD%92%E6%B1%82%E5%85%B7%E6%9C%89n%E4%B8%AA%E5%85%83%E7%B4%A0%E7%9A%84%E6%95%B4%E5%9E%8B%E6%95%B0%E7%BB%84r%E7%9A%84%E5%B9%B3%E5%9D%87%E5%80%BC/  

