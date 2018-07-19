---
title: 快排算法
date: 2015-05-22 10:29:14
tags: 算法
comments: true
---

### 图表演示
![算法演示](http://obakk2u63.bkt.clouddn.com/blog/quick-sort.png)

### Java实现
``` java
public class Test {
	
	public static void main(String[] args) {
		int[] arr = {1, 12, 5, 26, 7, 14, 3, 7, 2};
		
		System.out.print("排序前:");
		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i]);
			if (i != arr.length - 1) {
				System.out.print(",");
			}
		}
		System.out.print("\n");
		
		
		quickSort(arr, 0, arr.length -1);
		System.out.print("排序后:");
		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i]);
			if (i != arr.length - 1) {
				System.out.print(",");
			}
		}
		System.out.print("\n");
	}

	public static int partition(int arr[], int left, int right) {
		int i = left, j = right;
		int tmp;
		int pivot = arr[(left + right) / 2];
 
		while (i <= j) {
			while (arr[i] < pivot)
				i++;
			while (arr[j] > pivot)
				j--;
			if (i <= j) {
				tmp = arr[i];
				arr[i] = arr[j];
				arr[j] = tmp;
				i++;
				j--;
			}
		};
 
		return i;
	}
	 
	public static void quickSort(int arr[], int left, int right) {
		int index = partition(arr, left, right);
		if (left < index - 1) {
	    	  	quickSort(arr, left, index - 1);
		}
	      
		if (index < right) {
	    	  	quickSort(arr, index, right);
		}
	}
}
```

运行输出
``` shell
排序前:1,12,5,26,7,14,3,7,2
排序后:1,2,3,5,7,7,12,14,26
```
