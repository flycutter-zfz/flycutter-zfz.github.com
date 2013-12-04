---
layout: post
title: 排序-选择排序法
description: 排序算法，简单的排序算法之一
category: "algorithm"
tags: [排序]
---

##1. 算法描述

&nbsp;&nbsp;&nbsp;&nbsp;选择排序法也是一种简单的排序方式，每次遍历待排序的元素中最大（或最小）的元素，将其放在排序好的数列后，直到所有的数据排序完成。

##2. 代码

###递增排序
    {% highlight c++ %}
	void choose_sort_inc(int *a, int n){
		for(int i=0;i<n-1;i++){
			int min_index = i;
			for(int j=i+1; j<n; j++){
				if(a[min_index] > a[j]){
					min_index = j;
				}
			}
			int tmp = a[min_index];
			a[min_index] = a[i];
			a[i] = tmp;
		}	
	}
    {% endhighlight %}
###递减排序
	void choose_sort_dec(int *a, int n){
		for(int i=0;i<n-1;i++){
			int max_index = i;
			for(int j=i+1;j<n;j++){
				if(a[max_index]<a[j]){
					max_index = j;
				}
			}
			int tmp = a[max_index];
			a[max_index] = a[i];
			a[i] = tmp; 
		}
	}

##3. 算法分析

循环次数：(n-1)+(n-2)+...+1=n(n-1)/2，时间复杂度为n^2；
