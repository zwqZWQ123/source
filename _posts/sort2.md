---
title: 排序算法——简单选择排序、堆排序
date: 2017-04-20 10:01:05
tags:
- 简单选择排序
- 堆排序

description: 本篇博客讲述排序算法里的选择排序，具体有两种：简单选择排序和堆排序

categories: 排序算法
---
# 选择排序——简单选择排序
## 基本思想
&ensp;&emsp;&emsp;对于给定的一组记录，经过第一轮比较后得到最小的记录，然后将该记录与第一个记录的位置进行交换；接着对不包括第一个记录以外的其他记录进行第二轮比较，得到最小的记录并与第二个记录进行位置交换；重复该过程，直到进行比较的记录只有一个时为止。
简单选择排序示例：
![selectSort](/sort2/selectSort.png)
## 算法实现
```java
public static void selectSort(int[]a){
		int len = a.length;
		int temp = 0;
		int flag = 0;
		for (int i = 0; i < len; i++) {
			flag = i;
			temp = a[i];
			for (int j = i+1; j < len; j++) {
				if (a[j]<temp) {
					flag = j;
					temp = a[j];
				}
			}
			if (flag != i) {
				a[flag] = a[i];
				a[i] = temp;
			}
		}		
	}
```
# 选择排序——堆排序
## 基本思想
&ensp;&emsp;&emsp;堆排序是一种**树形选择排序**，是对直接选择排序的有效改进。合法的最大堆树要满足一个条件就是**每一个结点值都要大于或等于它的孩子结点值**。在一个数组中专业法表示为： arrays[i]>=arrays[2*i+1] && arrays[i]>=arrays[2*i+2] ; 最小堆类似，只要改为小于等于即可。
&ensp;&emsp;&emsp;堆排序树的构造过程找最大值过程由下图，数组arrays[0....n]为：17,8,45,84,2,94，刚找到最大值后把最大值即94放在数组的最后面arrays[n],然后进入递归把arrays[0...n-1]再进入下面图这个过程，只是把排好序的最大值不放入到这个过程中，就这样把值一个个的冒出来。找到最大值后把这个最大值放到数组的最后面，进入下一个递归。
![heapSort1](/sort2/heapSort1.gif)
&ensp;&emsp;&emsp;上图已经排好了最大那个值 下面见图排其他的元素：
![heapSort2](/sort2/heapSort2.gif)
&ensp;&emsp;&emsp;最后两步还有几个过程没画出来，最后两个图好像没有变化，但这里面还要排好几次，原因就是最后第二个图是不满足堆排序树的要调整后再把最大的值放到到后面。再次回到递归里面。
## 算法实现
```java
public static void heapSort(int array[],int e){
		if(e > 0){
			initHeap(array, e);
			array[0] = array[0] + array[e];
			array[e] = array[0] - array[e];
			array[0] = array[0] - array[e];
			heapSort(array,e-1);
		}
	}
	public static void initHeap(int array[],int e){
		int m = (e+1)/2;
		for (int i = 0; i < m; i++) {
			Boolean flag = buildHeap(array,e,i);
			if (flag) {
				i = -1;
			}
		}
		
	}
	public static Boolean buildHeap(int array[],int e,int i){
		int l_child = 2*i + 1;
		int r_child = 2*i + 2;
		if (r_child > e) {
			if (array[i] < array[l_child]) {
				array[i] = array[i] + array[l_child];
				array[l_child] = array[i] -array[l_child];
				array[i] = array[i] - array[l_child];
				return true;
			}else{
				return false;
			}
		}
		if (array[i] < array[l_child]) {
			if (array[l_child] < array[r_child]) {//交换根与右孩子的值
				array[i] = array[i] + array[r_child];
				array[r_child] = array[i] - array[r_child];
				array[i] = array[i] - array[r_child];
				return true;
			}else{//交换根与左孩子的值
				array[i] = array[i] + array[l_child];
				array[l_child] = array[i] -array[l_child];
				array[i] = array[i] - array[l_child];
				return true;
			}
		} else if(array[i] < array[r_child]){//交换根与右孩子的值
				array[i] = array[i] + array[r_child];
				array[r_child] = array[i] - array[r_child];
				array[i] = array[i] - array[r_child];
				return true;
			}
		return false;
	}
```



