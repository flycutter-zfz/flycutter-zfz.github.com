---
layout: post
title: 排序-插入排序
description: 排序算法，最基础的插入排序
category: "algorithm"
tags: [排序]
---

##1. 算法描述

&nbsp;&nbsp;&nbsp;&nbsp;插入排序是一种很简单的排序方法，类比于玩儿扑克牌时的情景，新抓的牌插入到合适的位置，保证手中的牌总是保持有序的状态。

##2. 代码

###递增排序
	void insert_sort_inc(int *a, int n){
		for(int i=1; i<n; i++){
			int key = a[i];
			int j = i-1;
			while(j>=0 && a[j]>key){
				a[j+1]=a[j];
				j--;
			}
			a[j+1] = key;
		}
	}

###递减排序
	void insert_sort_dec(int *a, int n){
		for(int i=1; i<n; i++){
			int key = a[i];
			int j = i - 1;
			while(j>=0 && a[j]<key){
				a[j+1]=a[j];
				j--;
			}
			a[j+1] = key;
		}
	}

##3. 算法分析

**最好情况** ：最好的情况是输入时已经按照所求的顺序排列，此时内部的循环总是第一次就跳出，所以时间复杂度为：n；

**最坏情况** ：最坏的情况是输入域所求的顺序相反，这样循环次数为：1+2+...+(n-1)=n(n-1)/2，时间复杂度为：n^2；