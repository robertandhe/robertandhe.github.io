---
layout: post
title: "Leetcode152-Maximum Product Subarray"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-11 16:15:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given an integer array nums, find the contiguous subarray within an array (containing at least one number) which has the largest product.
### Example 1:
```c++
Input: [2,3,-2,4]
Output: 6
Explanation: [2,3] has the largest product 6.
```
### Example 2:
```c++
Input: [-2,0,-1]
Output: 0
Explanation: The result cannot be 2, because [-2,-1] is not a subarray.
```

# 解法
## 解法 1
这题一看就是动态规划，但拿到手上感觉无法下手，先试试暴力法。:)
```c++
int maxProduct(vector<int>& nums)
{
	long long res = LLONG_MIN;
	int n = nums.size();
	for (int i = 0; i < n; ++i)
	{
		long long pro = 1;
		for (int j = i; j < n; ++j)
		{
			pro *= nums[j];
			res = max(res, pro);
		}
	}
	return (int)res;
}
```
time complexity: O(n^2)  
space complexity: O(1)  
runtime: 5.40%  
memory usage: 100.00%  

## 解法 2
接下来看看动态规划解法。这道题和最大子数组和很相似，那里只要遇到小于0就重新置起点。但在这里
遇到小于0可能以后还是会乘为正数。所以应该用两个动态数组存储最大值和最小值。mxVec[i]表示[0, i]范围内包括i位置的最大值，mnVec[i]表示[0, i]范围内包括i位置的最下值。那么状态转移方程为:
- ``mxVec[i] = max(max(mxVec[i - 1] * nums[i], mnVec[i - 1] * nums[i]), nums[i])``
- ``t = mxVec[i- 1]; mnVec[i] = min(min(t * nums[i], mnVec[i - 1] * nums[i]), nums[i])``

因为最终结果可能不在末尾，应该在中间更新。
```c++
int maxProduct(vector<int>& nums)
{
	if (nums.empty())
		return 0;
	int res = nums[0], n = nums.size();
	vector<int> mxVec(n, 0), mnVec(n, 0);
	mxVec[0] = nums[0];
	mnVec[0] = nums[0];
	for (int i = 1; i < n; ++i)
	{
		int t = mxVec[i - 1];
		mxVec[i] = max(max(mxVec[i - 1] * nums[i], mnVec[i - 1] * nums[i]), nums[i]);
		mnVec[i] = min(min(t * nums[i], mnVec[i - 1] * nums[i]), nums[i]);
		res = max(res, mxVec[i]);
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 38.34%  
memory usage: 47.50% 

## 解法 3
在解法2中，其实状态转移方程中更新是仅与上个状态有关，所以可以用两个变量mx和mn代替数组。
```c++
int maxProduct(vector<int>& nums)
{
	if (nums.empty())
		return 0;
	int res = nums[0], n = nums.size(), mx = nums[0], mn = nums[0];
	for (int i = 1; i < n; ++i)
	{
		int t = mx;
		mx = max(max(mx * nums[i], mn * nums[i]), nums[i]);
		mn = min(min(t * nums[i], mn * nums[i]), nums[i]);
		res = max(res, mx);
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 38.34%  
memory usage: 87.50%  

## 解法 4
把状态转移方程根据nums[i]大于0还是小于0改写。
```c++
int maxProduct(vector<int>& nums)
{
	if (nums.empty())
		return 0;
	int res = nums[0], mx = nums[0], mn = nums[0], n = nums.size();
	for (int i = 1; i < n; ++i)
	{
		if (nums[i] > 0){
			mx = max(mx * nums[i], nums[i]);
			mn = min(mn * nums[i], nums[i]);
		}
		else {
			int t = mx;
			mx = max(mn * nums[i], nums[i]);
			mn = max(t * nums[i], nums[i]);
		}
		res = max(res, mx);
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 89.08%  
memory usage: 87.50%  

## 解法 5
骚操作。把解法4中分情况改写到一个情形。如果nums[i]小于0，调换mx和mn值，其余相同。
```c++
int maxProduct(vector<int>& nums)
{
	if (nums.empty())
		return 0;
	int res = nums[0], mx = nums[0], mn = nums[0], n = nums.size();
	for (int i = 1; i < n; ++i)
	{
		if (nums[i] < 0){
			swap(mx, mn);
		}
		mx = max(mx * nums[i], nums[i]);
		mn = min(mn * nums[i], nums[i]);
		res = max(res, mx);
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 89.08%  
memory usage: 92.50%  

## 解法 6
别出新意。两轮遍历，一轮从前往后，一轮从后往前。在遍历中，出现累乘积等于0时，要重置为1。
```c++
int maxProduct(vector<int>& nums)
{
	if (nums.empty())
		return 0;
	int res = nums[0], prod = 1, n = nums.size();
	for (int i = 0; i < n; ++i)
	{
		prod *= nums[i];
		res = max(res, prod);
		if (prod == 0)
			prod = 1;
	}
	prod = 1;
	for (int i = n - 1; i >= 0; --i)
	{
		prod *= nums[i];
		res = max(res, prod);
		if (prod == 0)
			prod = 1;
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 38.34%  
memory usage: 65.00%  

# Link
[leetcode-152](https://leetcode.com/problems/maximum-product-subarray/)  
[solution](https://www.cnblogs.com/grandyang/p/4028713.html)  