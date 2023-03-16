# 稀疏数组


<!--more-->


## 介绍
当一个数组中大部分元素为0，或者为同一值的数组时，可以使用稀疏数组来保存该数组。


稀疏数组的处理方式是：
- 记录数组一共有几行几列，有多少个不同值
- 把具有不同值的元素和行列及值记录在一个小规模的数组中，从而缩小程序的规模


如下图：
![稀疏矩阵](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//20230316195733.png)




## 例子
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//20230316195809.png)

那么这个就是：
11,11,2
1,2,1
2,3,2  第二行第三列值是2


```java

package basic;

/**
 * @author 剑胆琴心
 * @create 2023-03-16 16:24
 */
public class Practice1 {
    public static void main(String[] args) {

        System.out.println("输出原始数组:");
        //1.创建一个二维数组11*11，,0表示没有棋子，1：黑棋，2白棋
        int[][] arraryy = new int[11][11];
        arraryy[1][2]=1;
        arraryy[2][3]=2;
        //输出原始数组
        for(int[] ins:arraryy){
            for(int intA: ins){
                System.out.print(intA +"\t");
            }
            System.out.println();
        }

        System.out.println("------");
        //转换为稀疏数组保存
        //获取有效值的个数
        int summ = 0;
        for(int i=0; i< 11; i++){
            for(int j=0; j<11; j++){
                if (arraryy[i][j] !=0){
                    summ++;
                }
            }
        }
        System.out.println("有效值个数：" + summ);

        //2.创建一个稀疏数组的数组
        int[][] arr_xishu = new int[summ+1][3];
        arr_xishu[0][0] = 11;
        arr_xishu[0][1] = 11;
        arr_xishu[0][2] = summ;  //稀疏数组的头打印完毕

        //遍历二维数组，将非0值存放到稀疏数组中
        int count=0;
        for(int i=0; i< arraryy.length; i++){
            for(int j=0; j<arraryy[i].length; j++){
                if (arraryy[i][j] !=0){
                    count++; //3.
                    // if有效数字的话会先count++变成1，所以是从第二行开始存放，不影响有几个有效数字代表count是几
                    // 可以理解成第一个有效数字的i、j坐标和数值放在了第一个count（这时候是等于1）那行
                    arr_xishu[count][0] = i;  //第几行，第0列存放横坐标
                    arr_xishu[count][1] = j;  //第几行，第1列存放横坐标
                    arr_xishu[count][2] = arraryy[i][j];  //第几行，第2列存放值

                }
            }
        }
        System.out.println("------输出稀疏数组：");
        //输出稀疏数组
        for(int i=0; i<arr_xishu.length; i++){
            System.out.println(arr_xishu[i][0] +"\t"
                    + arr_xishu[i][1] +"\t"
                    + arr_xishu[i][2] +"\t"
            );
        }

        //还原稀疏数组
        //1.读取稀疏数组
        int[][] arr3 = new int[arr_xishu[0][0]][arr_xishu[0][1]];
        //2.还原值
        for(int i=1; i< arr_xishu.length;i++){
            arr3[arr_xishu[i][0]][arr_xishu[i][1]] = arr_xishu[i][2];
        }
        System.out.println("------稀疏矩阵还原：");
        //打印
        for(int[] ins:arraryy){
            for(int intA: ins){
                System.out.print(intA +"\t");
            }
            System.out.println();
        }



    }
}

```

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//20230316200524.png)















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E7%A8%80%E7%96%8F%E6%95%B0%E7%BB%84/  

