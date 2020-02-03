---
layout: post
title: "Leetcode452-Minumum Number of Arrow to Burst Ballons"
subtitle: 'Leetcode452-greedy'
date:       2019-10-17 17:30:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
---

# 问题描述
There are a number of spherical balloons spread in two-dimensional space. For each balloon, provided input is the start and end coordinates of the horizontal diameter. Since it's horizontal, y-coordinates don't matter and hence the x-coordinates of start and end of the diameter suffice. Start is always smaller than end. There will be at most 104 balloons.

An arrow can be shot up exactly vertically from different points along the x-axis. A balloon with xstart and xend bursts by an arrow shot at x if xstart ≤ x ≤ xend. There is no limit to the number of arrows that can be shot. An arrow once shot keeps travelling up infinitely. The problem is to find the minimum number of arrows that must be shot to burst all balloons.  
#### Example:
```c++
Input:
[[10,16], [2,8], [1,6], [7,12]]

Output:
2

Explanation:
One way is to shoot one arrow for example at x = 6 (bursting the balloons [2,8] and [1,6]) and another arrow at x = 11 (bursting the other two balloons).
```

# 解法
## 解法1
题目给定区间表示气球位置，从x轴某个位置垂直射箭，一条线上的气球均可击爆。为较简单的贪心问题。
先给数组排序，位置靠前的在前。用一个stack st存放上一可射中区间。每遍历到一个气球，用气球起点和上一区间末尾短点比较可知道当前
气球是否在上一可中范围内，如果不在则要再来一发。如果在的话就要更新区间，有区间的交集来更新。
```c++
int findMinArrowShots(vector<vector<int>>& points) 
{
	if (points.empty())
		return 0;
	stack<vector<int>> st;
	sort(points.begin(), points.end());
	st.push(points[0]);	
	for (int i = 1; i < points.size(); ++i)
	{
		if (points[i][0] > st.top()[1]){
			st.push(points[i]);
			continue;
		}
		else {
			int a = max(st.top()[0], points[i][0]);
			int b = min(st.top()[1], points[i][1]);
			st.top() = {a, b};
		}
	}
	return st.size();
}
```
时间复杂度：O(nlogn)  
空间复杂度：O(n)  
runtime : 82.55%   memory usage: 40.00%
## 解法2
和解法1思想一样，用一个数组pre存放上一区间，不停更新它。
```c++
int findMinArrowShots(vector<vector<int>>& points) 
{
	if (points.empty())
		return 0;
	int res = 0, n = points.size();
	sort(points.begin(), points.end());		
	vector<int> pre = points[0];
	++res;
	for (int i = 1; i < n; ++i)
	{
		if (points[i][0] > pre[1]){
			++res;
			pre = points[i];
		}
		else {
			pre[0] = max(pre[0], points[i][0]);
			pre[1] = min(pre[1], points[i][1]);
		}
	}
	return res;
}
```
时间复杂度：O(nlogn)  
空间复杂度：O(n)  
runtime : 53.12%   memory usage: 100.00%
## 解法2
和解法1，2思想一样，只是区间真正有用的是末尾端点，只需要存放末尾点end就行了。
```c++
int findMinArrowShots(vector<vector<int>>& points) 
{
	if (points.empty())
		return 0;
	int res = 0, end = 0, n = points.size();
	sort(points.begin(), points.end());
	end = points[0][1];
	++res;
	for (int i = 1; i < n; ++i)
	{
		if (points[i][0] <= end){
			end = min(end, points[i][1]);
		}
		else {
			++res;
			end = points[i][1];
		}
	}
	return res;
}
```
时间复杂度：O(nlogn)  
空间复杂度：O(n)  
runtime : 53.12%   memory usage: 100.00%

# link
[leetcode-452](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)  
[solution 3](https://www.cnblogs.com/grandyang/p/6050562.html)