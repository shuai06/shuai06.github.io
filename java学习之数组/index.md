# Java学习之数组




### 数组

- 数组(Array)，是多个**相同类型数据**按**一定顺序排列**的集合，并使用**一个名字命名**，并通过**编号**的方式对这些数据进行统一管理。
- 数组本身就是**引用数据类型**，而数组中的元素可以是**任何数据类型**。
- 创建数组对象会在内存中开辟一整块**连续的空间**，而数组名中引用的是这块连续空间的**首地址**。
- 数组的**长度一旦确定，就不能修改**。



### 一维数组

#### 使用

```java
public class ArrayTest {
	public static void main(String[] args) {
		
		int[] ids; // 声明
		ids = new int[] {1001, 1002, 1003, 1004, 1005, 1006}; //初始化方式1(静态初始化)：数组的初始化和数组元素的赋值操作同时进行
		
		String[] names = new String[5]; //动态初始化：数组的初始化和数组元素的赋值操作分开进行
		//一旦初始化完毕，数组长度就确定了
		int[] ididids ={1001, 1002, 1003, 1004, 1005, 1006}; //类型推断(非标准写法)
		
		//错误写法...
		//int[] arr1 = new int[];
		//int[5] arr1 = new int[];
		//int[] arr1 = new int[3]{1,2,3};
		
		
		
		
		//2.调用指定位置的元素: 通过索引[0,n-1]
		names[0] = "张三";  // 通过角标赋值
		names[1] = "陈奕迅";
		names[2] = "朱一龙";
		names[3] = "张杰";
		names[4] = "伍佰";  // charAt(0)，只要不是数据库交互，都是从0开始
		System.out.print(names[1]);
		
		
		
	    //3.如何获取数组长度: .length属性
		System.out.print(names.length);
		System.out.println(ids.length);
		
		
		
		 //4.如何遍历数组
		for(int i=0; i < names.length; i++) {  //最大到长度-1
			System.out.println("姓名：" + names[i] + "\t学号：" + ids[i]);
		}
		
		
		
		 //5.数组元素的默认初始化值
		// 若数组元素是整型：默认初始化值为0
		//  ...浮点型：0.0
		// 若数组元素是char：默认初始化值为0 ,形式上为"空格"
		//  ...布尔型：false
		// ... 引用数据类型时(比如String)：null
		int[] arr1 = new int[4];
		for (int i = 0; i < arr1.length; i++) {
			System.out.println(arr1[i]);
		}
		// 若数组元素是String：默认初始化值为null
		String[] arr2 = new String[4];
		for (int i = 0; i < arr2.length; i++) {
			System.out.println(arr2[i]);
		}
		
		
		 //6.一维数组的内存解析

		
		
		System.out.println("111");
		// 
	}
}

```

#### 一维数组的内存解析

**首先，在jvm中，内存的简化结构如下:**

![](http://image.xpshuai.cn/%E7%AE%80%E5%8C%96%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.jpg)



**一维数组的内存解析如下：**

![](http://image.xpshuai.cn/%E4%B8%80%E7%BB%B4%E6%95%B0%E7%BB%84%E7%9A%84%E5%86%85%E5%AD%98%E8%A7%A3%E6%9E%90.png)



**引用计数算法：**当栈中没有变量指向堆空间的地址，就会自动回收这段堆中的垃圾地址



### 二维数组

对于二维数组，可以看做是一维数组又作为另一个一个数组的元素而存在。

其实，从数组底层的运行机制来看，并没有多维数组。

#### 使用

```java
public class DoubleArrayTest {
	public static void main(String[] args) {
		//* 1.声明和初始化
		//二维
		//静态初始化
		int[][] arr2 = new int[][]{{1,2,3}, {4,5,6}, {7,8,9}};
		// 动态初始化
		String[][] arr22 = new String[3][2]; // 行列
		String[][] arr222 = new String[3][];
		String arr2222[][] = new String[3][4];
		int[][] arr22222 ={{1,2,3}, {4,5,6}, {7,8,9}};
		String[][] names = new String[4][2];
		
		//错误情况：
		//String[][] arr22 = new String[][4];
		
		
		//2.调用指定位置的元素
		names[0][0] = "张三";  // 通过角标赋值
		names[0][1] = "陈奕迅";
		names[1][0] = "朱一龙";
		names[1][1] = "张杰";
		names[2][0] = "伍佰";  
		names[2][1] = "迪丽热巴";  
		names[3] = new String[2]; //
		//System.out.println(names[1][0]);
		for (int i = 0; i < names.length; i++) {
			for (int j = 0; j < names[i].length; j++) {
				System.out.print(names[i][j] + "\t"); //未赋值的为null
			}
			System.out.println();
		}
		
		
		
	    //3.如何获取数组长度: .length属性
		System.out.println(names.length);  //获取的是外层数组的长度
		System.out.println(names[0].length); // 获取的外层第一个元素的数组的长度
		System.out.println(names[1].length);
		
		
		
		//4.如何遍历二维数组：两层（先遍历指定行再遍历列）
		for (int i = 0; i < names.length; i++) {
			for (int j = 0; j < names[i].length; j++) {
				System.out.print(names[i][j] + "\t"); //未赋值的为null
			}
			System.out.println();
		}

		
		
		//5.二位数组元素的默认初始化值
		// 外层元素(是一个地址值)，内层元素(同一维)
		int[][] arrDouble = new int[4][3];
		System.out.println(arrDouble);  // 二维的地址值：[[I@15db9742
		System.out.println(arrDouble[0]);  //外层是一个地址值   [I@6d06d69c
		System.out.println(arrDouble[0][0]); // 是0
		
		String[][] arrDoubleS = new String[4][3];
		System.out.println(arrDoubleS[0]);  //是一个地址值
		System.out.println(arrDoubleS[0][0]); // 是null

		
		
		
		 //6.二维数组的内存解析
	}
}

```



#### 二维数组的内存解析

![](http://image.xpshuai.cn/%E4%BA%8C%E7%BB%B4%E6%95%B0%E7%BB%84%E5%86%85%E5%AD%98%E8%A7%A3%E6%9E%901.jpg)

![](http://image.xpshuai.cn/%E4%BA%8C%E7%BB%B4%E6%95%B0%E7%BB%84%E5%86%85%E5%AD%98%E8%A7%A3%E6%9E%902.jpg)







### Arrays工具类

```java
public class ArraysToolClassTest {
	public static void main(String[] args) {
		// boolean equals(int[] a,int[] b)
		int[] arr1 = new int[]{1,2,3};
		int[] arr2 = new int[] {10,3,9};
		boolean isEquals = Arrays.equals(arr1, arr2);
		System.out.println(isEquals);
		
		
		// String toString(int[] a)  输出数组信息
		System.out.println(Arrays.toString(arr1));
		
		
		// void fill(int[] a, int val)  将指定值填充到数组中
		Arrays.fill(arr1, 9);
		System.out.println(Arrays.toString(arr1));
		
		
		// void sort(int[] a) 
		Arrays.sort(arr2);
		System.out.println(Arrays.toString(arr2));
		
		
		// int binarySearch(int[] a, int key)  二分查找(前提：数组必须有序)
		int[] arr3 = new int[] {-98,-50,-3,0,3,6,99,210,999};
		int index = Arrays.binarySearch(arr3, 210);  //如果找到了返回的是索引，如果找不到就返回负数
		System.out.println(index);
	}
}

```







---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E6%95%B0%E7%BB%84/  

