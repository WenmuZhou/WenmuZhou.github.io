---
title: 2017CVTE-C++笔试题-求最大和子序列
date: 2017/03/26 20:27:00
comments: true
categories: 
- 笔试题
tags: 
- 笔试题
- C++
---

# 题目
> 输入一个整型数组，数组里有正数也有负数。 数组中连续的一个或多个整数组成一个子数组，每个子数组都有一个和。 求所有子数组的和的最大值。要求时间复杂度为O(n)。 例如输入的数组为1, -2, 3, 10, -4, 7, 2, -5，和最大的子数组为3, 10, -4, 7, 2，

# 代码
```C++
#include<iostream>
using namespace std;

#define NUM 8

int main()
{
	int ary[NUM] = { 1, -2, 3, 10, -4, 7, 2, -5 };
	
	int max = 0;//保存最大和
	int curSum = 0;//保存当前和
	int curStart = 0;//当前和的起始位置
	int start = 0;//最大和的起始位置
	int end = 0;//最大和的终止位置
	for (int i = 0; i<NUM; i++)
	{
		if (i == 0)
		{
			curSum = max = ary[i];
			continue;
		}

		if (curSum<0)
		{
			curSum = 0;//与负数相加，和会减小，所以抛弃以前的和
			curStart = i;
		}
		//最大值已经被保存下来，所以请大胆的继续往前加
		curSum += ary[i];
		//当前和被保存为最大值，记录下它的起始位置和结束位置
		if (curSum>max)
		{
			max = curSum;
			start = curStart;
			end = i;
		}
	}

	cout << "和最大的子数组为：" << endl;
	for (int i = start; i <= end; i++)
	{
		cout << ary[i] << " ";
	}
	cout << "= " << max;
	cin.get();
    return 0;
}
```

# 结果

![结果.png](http://upload-images.jianshu.io/upload_images/1575688-1baba445c002f9a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
