---
title: 排序算法——冒泡排序、快速排序
date: 2017-04-20 10:42:48
tags:
- 冒泡排序
- 快速排序

description: 本篇博客讲述排序算法里的交换排序，具体有两种：冒泡排序和快速排序

categories: 排序算法
---
# 冒泡排序
## 基本思想
&ensp;&emsp;&emsp;冒泡排序顾名思义就是整个过程就像气泡一样往上升，单向冒泡排序的基本思想是（假设由小到大排序）：对于给定的n个记录，从第一个记录开始依次对相邻的两个记录进行比较，当前面的记录大于后面的记录时，交换位置，进行一轮比较和换位后，n个记录中的最大记录将位于第n位；然后对前(n-1)个记录进行第二轮比较；重复该过程直到进行比较的记录只剩下一个为止。
冒泡排序的示例：
![bubbleSort](/sort3/bubbleSort.jpg)
## 算法实现
```java
public static void bubbleSort(int[] a){
		int temp = 0;
		for (int i = a.length; i > 0; i--) {
			for (int j = 1; j < i; j++) {
				if (a[j] < a[j-1]) {
					temp = a[j-1];
					a[j-1] = a[j];
					a[j] = temp;
				}
			}
		}
	}
```

# 快速排序
## 基本思想
1. 选择一个基准元素,通常选择第一个元素或者最后一个元素。
2. 通过一趟排序讲待排序的记录分割成独立的两部分，其中一部分记录的元素值均比基准元素值小。另一部分记录的 元素值比基准值大。
3. 此时基准元素在其排好序后的正确位置。
4. 然后分别对这两部分记录用同样的方法继续进行排序，直到整个序列有序。

快速排序的示例：
（a）一趟排序的过程：
![quickSort1](/sort3/quickSort1.jpg)
（b）排序的全过程
![quickSort2](/sort3/quickSort2.jpg)
&ensp;&emsp;&emsp;快速排序是通常被认为在**同数量级（O(nlog2n)）**的排序方法中平均性能最好的。但若初始序列按关键码有序或基本有序时，快排序反而蜕化为冒泡排序。为改进之，通常以“三者取中法”来选取基准记录，即将排序区间的两个端点与中点三个记录关键码居中的调整为支点记录。**快速排序是一个不稳定的排序方法。**
## 算法实现
```java
public static void sort(int array[],int low,int high){
		int i,j;
		int index;
		if (low >= high) {
			return;
		}
		i = low;
		j = high;
		index = array[i];
		while (i < j) {
			while (i<j && array[j] >= index) {
				j--;
			}
			if (i<j) {
				array[i++] = array[j];
			}
			while (i<j && array[i] <= index) {
				i++;
			}
			if (i<j) {
				array[j--] = array[i];
			}
		}
		array[i] = index;
		sort(array,low,i-1);
		sort(array,i+1,high);
	}
	public static void quickSort(int array[]){
		sort(array, 0, array.length-1);
	}
```



