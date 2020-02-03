---
layout: post
title: "Leetcode84-Largest Rectangle in Histogram "
subtitle: 'Leetcode84-stack'
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - stack
---
# 问题描述：
Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.
![81-1](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/leetcode/84-1.PNG)  
Above is a histogram where width of each bar is 1, given height = [2,1,5,6,2,3].
![81-2](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/leetcode/84-2.PNG)  
The largest rectangle is shown in the shaded area, which has area = 10 unit.  
Example:
```c++
Input: [2,1,5,6,2,3]
Output: 10
```
# 分析：
题目给出一个表示直方图的向量，要求出直方图内最大的矩形面积。对求面积来说，相当于水桶原理，矮的边对矩形面积形成限制。
暴力法可以求解，但无法通过OJ。

# 解法1：局部峰值
如例子中1，5，6来说，如果5能形成矩形的右边，则右边大于它的6肯定也能形成矩形的右边，且面积更大，因此应该跳过1，5。
```c++
int largestRectangleArea(vector<int>& heights) 
{
	int res = 0, mn = 0;
	for (int i = 0; i < heights.size(); ++i)
	{
		while (i + 1 < heights.size() && heights[i] <= heights[i + 1])
			++i;
		mn = heights[i];
		for (int j = i; j >= 0; --j)
		{
			mn = min(mn, heights[j]);
			int t = mn * (i - j + 1);
			res = max(res, t);
		}
	}
	return res;
}
```

# 解法2：单调栈
单调栈的好处就是只用遍历一次数据。再决定用递增栈还是递减栈，在这个问题中，短边形成对矩形面积的限制，当出现短边是才开始处理，短边
属于触发信号，因此应维护递增栈。大于栈顶元素出现时，矩形面积肯定在增大，直接推入栈，小于栈顶元素出现时开始判断，对递增栈内每一元素，后面的肯定比前面大，因此对每一栈顶，短边就是栈顶值，高度已经确定，宽度就是栈顶元素位置到当前元素位置的差距。因此栈内应该存放元素位置而不是元素值。
```c++
int largestRectangleArea(vector<int>& heights) 
{
	int res = 0;
	heights.push_back(0);
	stack<int> st;
	int i = 0, n = heights.size();
	while (i < n){
		while (!st.empty() && heights[i] <= heights[st.top()]){
			int cur = st.top();
			st.pop();
			res = max(res, heights[cur] * (st.empty() ? i : (i - st.top() - 1)));
		}
		st.push(i++);
	}
	return res;
}
```
在上面程序中，之所以出现``i - st.top() - 1``是因为对原始向量的元素推入递增栈后，后面小的元素会把前面打的元素pop。
所以在元素中间有峰顶导致位置间断时，栈顶元素位置肯定是左边。如果用``i - cur``可能会导致跳过已经pop掉的高的边，博主
开始也没注意这点。