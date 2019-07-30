---
title: 排序算法
date: 2017-05-23 20:54:18
tags:
 - 算法
 - 排序
 - Java
 - C
categories: [算法]
---

# 冒泡排序
## 思想
比较相邻记录关键字，如果逆序，则进行交换，从而使记录小的像气泡一样逐渐往上“漂浮”(左移)，或者说使记录大的像石头那样"下沉"(右移)

## 算法实现
### java实现
原序列
```java
int a[10] = {4,3,2,6,1,9,5,8,7,0};
```

```java
    private static void popSort(int[] a){
        //最基本的冒泡排序写法
        /*int i;
        for (i=0;i<a.length;i++){
            for (int j=0;j<a.length-1;j++){
                if (a[j]>a[j+1]){
                    int temp = a[j];
                    a[j] = a[j+1];
                    a[j+1] = temp;
                }
            }
        }*/

        //改进的冒泡排序写法
        int len = a.length-1;
        boolean flag = true;//用来标识是否发生交换操作
        while (len>0 && flag){
            flag = false;
            for (int j=0;j<len;j++){
                if (a[j] > a[j+1]){
                    //交换
                    int temp = a[j];
                    a[j] = a[j+1];
                    a[j+1] = temp;
                    flag = true;
                }
            }
            show(a);
            len--;
        }
    }
```

运行结果
```
原序列: 1 6 5 9 3 8 2 7 0 4 
1 5 6 3 8 2 7 0 4 9 
1 5 3 6 2 7 0 4 8 9 
1 3 5 2 6 0 4 7 8 9 
1 3 2 5 0 4 6 7 8 9 
1 2 3 0 4 5 6 7 8 9 
1 2 0 3 4 5 6 7 8 9 
1 0 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
排序后: 0 1 2 3 4 5 6 7 8 9 
```

# 快速排序(冒泡排序升级版)
## 思想
在待排记录中选择一个记录(通常第一个)作为枢纽(关键记录)pivotkey. 经过一趟排序，以枢纽记录分为两个子表，左边子表记录比枢纽记录小，右边子表比枢纽记录大。然后对每个子表重复以上过程。知道每个子表只有一个记录时，排序结束。
## 每一趟操做
 参考《大话数据结构》有详细解析
 1. 选择表中一个记录作为枢纽(通常是第一个记录) 保存好记录的值pivotkey
 2. (1)从high往左每个记录依次与pivotkey比较，比pivotkey大或相等的，high往左移(high--), 比pivotkey小的结束high移动，交换low和high的值
 	(2) 从low往右每个记录依次与pivotkey比较，比pivotkey小或相等的，low往右移(low++), 比pivotkey小的结束low移动，交换low和high的值
 3. 返回pivotkey的值
## 算法实现
### java实现
```
int a[10] = {4,3,2,6,1,9,5,8,7,0};
```

```
    private static void quickSort(int[] a,int low, int high){
        if (low<high){
            int povit = partition(a,low,high);
            quickSort(a,povit+1,high);  //对枢纽记录右边子表进行排序
            quickSort(a,low,povit-1);   //对枢纽记录左边的子表进行排序
        }
    }

    private static int partition(int[] a, int low, int high){
        System.out.println("low = " + low);
        int pivot = a[low]; //用子表的第一个记录作为枢纽记录
        while (low<high){   //
            while (low<high && a[high]>=pivot)  //右往左比较
                high--;                         //如果记录比枢纽记录大或者相等，high向左移动一个
            swap(a,low,high);                   //交换，比枢纽记录小的放左边
            show(a);
            while (low<high && a[low]<=pivot)   //左往右比较
                low++;                          //如果记录比枢纽记录小或者相等，high向右移动一个
            swap(a,low,high);                   //交换，比枢纽记录大的放右边
            show(a);
        }
        System.out.println("pivot = " + pivot);
        System.out.println();
        return low;
    }
```


调用
```java
quickSort(a, 0, a.length-1);
```

运行结果
```java
原序列: 1 6 5 9 3 8 2 7 0 4 
low = 0
0 6 5 9 3 8 2 7 1 4 
0 1 5 9 3 8 2 7 6 4 
0 1 5 9 3 8 2 7 6 4 
0 1 5 9 3 8 2 7 6 4 
pivot = 1

low = 2
0 1 4 9 3 8 2 7 6 5 
0 1 4 5 3 8 2 7 6 9 
0 1 4 2 3 8 5 7 6 9 
0 1 4 2 3 5 8 7 6 9 
0 1 4 2 3 5 8 7 6 9 
0 1 4 2 3 5 8 7 6 9 
pivot = 5

low = 6
0 1 4 2 3 5 6 7 8 9 
0 1 4 2 3 5 6 7 8 9 
pivot = 8

low = 6
0 1 4 2 3 5 6 7 8 9 
0 1 4 2 3 5 6 7 8 9 
pivot = 6

low = 2
0 1 3 2 4 5 6 7 8 9 
0 1 3 2 4 5 6 7 8 9 
pivot = 4

low = 2
0 1 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
pivot = 3

排序后: 0 1 2 3 4 5 6 7 8 9 
```

# 选择排序
## 思想
一个简单的选择排序算法是这样的:   
首先, 找到数组中最小的一个元素  
其次, 将它和数组的第一个元素交换位置,(如果第一个元素是最小的那么它就和自己交换).  
再次, 在剩下元素中继续找到最小的元素,将它和数组中第二个元素交换位置.  
如此反复操作,直到将整个数组排序  
这种方法就是简单选择排序.

## 算法实现
### C语言
原序列
```
int a[10] = {4,3,2,6,1,9,5,8,7,0};
```

```
/**简单选择排序
 * */
void SelectSort(int a[], int length){
    int i, j;
    int min;
    for (i = 0; i < length; ++i) {
        min = i;
        for (j = i+1; j < length; ++j) {
            if (a[j] < a[min]){
                min = j;
            }
            //printf("min = %d\n", min);//调试加上,
        }
        if (min != i){
            int temp = a[i];
            a[i] = a[min];
            a[min] = temp;
        }
        show(a,length);
    }
}
```
运行结果:
```java
原序列: 4 3 2 6 1 9 5 8 7 0 
0 3 2 6 1 9 5 8 7 4 
0 1 2 6 3 9 5 8 7 4 
0 1 2 6 3 9 5 8 7 4 
0 1 2 3 6 9 5 8 7 4 
0 1 2 3 4 9 5 8 7 6 
0 1 2 3 4 5 9 8 7 6 
0 1 2 3 4 5 6 8 7 9 
0 1 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
排序后: 0 1 2 3 4 5 6 7 8 9
```
### java实现
原序列:
```
int[] a = {1,6,5,9,3,8,2,7,0, 4};
```
第一次从0-9中选择最小的与第0个交换
第二次从1-9中选择最小的与第1个交换
重复以上步骤
```java
    private static void selectSort(int[] a){
        System.out.println("选择排序算法:");
        int min, i, j;
        for (i=0; i<a.length; i++){
            min = i;
            for (j=i+1; j<a.length; j++){
                if (a[j] < a[min]){
                    min = j;
                }
            }
            //System.out.println("min = " + min);
            int temp = a[i];
            a[i] = a[min];
            a[min] = temp;
            System.out.print("第" + (i+1) + "次排序后:");
            show(a);
        }
    }
```
运行结果:
```java
原序列: 1 6 5 9 3 8 2 7 0 4 
选择排序算法:
第1次排序后:0 6 5 9 3 8 2 7 1 4 
第2次排序后:0 1 5 9 3 8 2 7 6 4 
第3次排序后:0 1 2 9 3 8 5 7 6 4 
第4次排序后:0 1 2 3 9 8 5 7 6 4 
第5次排序后:0 1 2 3 4 8 5 7 6 9 
第6次排序后:0 1 2 3 4 5 8 7 6 9 
第7次排序后:0 1 2 3 4 5 6 7 8 9 
第8次排序后:0 1 2 3 4 5 6 7 8 9 
第9次排序后:0 1 2 3 4 5 6 7 8 9 
第10次排序后:0 1 2 3 4 5 6 7 8 9 
排序后: 0 1 2 3 4 5 6 7 8 9 
```

# 插入排序
## 思想
和整理扑克牌一样, 将每一张牌插入到其他已经有序的牌中的适当位置.在计算机实现中,为了要给插入的元素腾出空间, 我们需要将其余所有元素在插入之前都向右移动一位. 这种算法叫做插入排序.

## 实现
### C语言实现
原序列
```c
int a[10] = {4,3,2,6,1,9,5,8,7,0};
```


```c
/**
 * 直接插入排序*/
void InsertSort(int a[], int length){
    int i,j;
    for (i = 0; i < length- 1; ++i) {
        for (j = i+1; j > 0 ; j--) {
            if (a[j] < a[j-1]){
                //交换
                int temp = a[j];
                a[j] = a[j-1];
                a[j-1] = temp;
            }
        }
        show(a,MAX_SIZE);
    }
}
```
运行结果:
```
原序列: 4 3 2 6 1 9 5 8 7 0 
3 4 2 6 1 9 5 8 7 0 
2 3 4 6 1 9 5 8 7 0 
2 3 4 6 1 9 5 8 7 0 
1 2 3 4 6 9 5 8 7 0 
1 2 3 4 6 9 5 8 7 0 
1 2 3 4 5 6 9 8 7 0 
1 2 3 4 5 6 8 9 7 0 
1 2 3 4 5 6 7 8 9 0 
0 1 2 3 4 5 6 7 8 9 
排序后: 0 1 2 3 4 5 6 7 8 9
```

### Java实现
原序列:
```java
int[] a = {1,6,5,9,3,8,2,7,0,4};
```
第一次把第2个数依次从右往左和1个数比较
第二次把第3个数依次从右往左和2个数比较
重复以上步骤

```java
    private static void insertSort(int[] a){
        System.out.println("插入排序算法:");
        int i, j;
        for (i=0; i<a.length; i++){
            for (j=i; j>0; j--){
                if (a[j] < a[j-1]){
                    int temp = a[j];
                    a[j] = a[j-1];
                    a[j-1] = temp;
                }
            }
            System.out.print("第" + (i+1) + "次排序后:");
            show(a);
        }
    }
```
运行结果
```java
原序列: 1 6 5 9 3 8 2 7 0 4 
插入排序算法:
第1次排序后:1 6 5 9 3 8 2 7 0 4 
第2次排序后:1 6 5 9 3 8 2 7 0 4 
第3次排序后:1 5 6 9 3 8 2 7 0 4 
第4次排序后:1 5 6 9 3 8 2 7 0 4 
第5次排序后:1 3 5 6 9 8 2 7 0 4 
第6次排序后:1 3 5 6 8 9 2 7 0 4 
第7次排序后:1 2 3 5 6 8 9 7 0 4 
第8次排序后:1 2 3 5 6 7 8 9 0 4 
第9次排序后:0 1 2 3 5 6 7 8 9 4 
第10次排序后:0 1 2 3 4 5 6 7 8 9 
排序后: 0 1 2 3 4 5 6 7 8 9 
```

# 希尔排序
## 思想
希尔排序的思想是使数组中任意间隔为h的元素都是有序的. 这样的数组成为h有序数组  
简单地说, 一个h有序数组就是h个互相独立的有序数组编织在一起组成的一个数组.  
在进行排序时, 如果h很大, 我们能将元素移动到很远的地方, 为实现更小的h有序数组创造方便, 用这种方式,对于任意以1结尾的h序列, 我们都能够将数组排序.  
这就是希尔排序.

##　实现
### C语言实现
```c
/**
 * 希尔排序*/
 void ShellSort(int a[], int length){
    int h = 1;  //增量
    //while(h <= length/3) h = 3*h + 1;   //增量初始值 我写的
    while(h < length/3) h = 3*h + 1;   //增量初始值

    while (h >= 1){     //对每个增量都进行排序
        int i;
        for (i = h; i < length; ++i) {      //对同一增量的每个分组进行排序
            int j;
            //for (j = i; j < length; j= j-h) {   //对每个分组进行插入排序     我写的
            for (j = i; j >= h; j= j-h) {   //对每个分组进行插入排序
                if(a[j] < a[j-h]){
                    int temp = a[j];
                    a[j] = a[j-h];
                    a[j-h] = temp;
                }

            }
        }
        show(a,MAX_SIZE);
        h = h/3;
    }
}
```

运行结果
```c
原序列: 4 3 2 6 1 9 5 8 7 0 
1 0 2 6 4 3 5 8 7 9 
0 1 2 3 4 5 6 7 8 9 
排序后: 0 1 2 3 4 5 6 7 8 9 
```

### Java实现
```java
    private static void shellSort(int[] a){
        int N = a.length;
        int h = 1;
        while (h < (N/3)) h = h * 3 + 1;
        while (h >= 1){
            for (int i=h; i<N; i++){
                for (int j=i; j>=h;j=j-h){
                    //进行插入排序
                    if (a[j] < a[j-h]){
                        int temp = a[j];
                        a[j]= a[j-h];
                        a[j-h] = temp;
                    }
                }
            }
            System.out.print("h = " + h + ": ");
            show(a);
            h = h/3;
        }
    }
```
运行结果:
```java
原序列: 1 6 5 9 3 8 2 7 0 4 
h = 4: 0 4 2 7 1 6 5 9 3 8 
h = 1: 0 1 2 3 4 5 6 7 8 9 
排序后: 0 1 2 3 4 5 6 7 8 9 
```