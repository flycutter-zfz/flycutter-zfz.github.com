---
layout: post
title: "Funny Programs"
description: ""
category: ""
tags: []
---

###1. 统计一个int值的二进制表达式中1的个数
{% highlight c++ %}
int func(int n){
	int count=0;
	while(n){
		++count;
		n=n&(n-1);
	}
	return count;
}

int main(){
	cout << func(9999) << endl;      //输出8，二进制表达式中有8个1
	return 0;
}
{% endhighlight %}
