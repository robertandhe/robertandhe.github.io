---
layout: post
title: "Leetcode978 - Longest Turbulent Subarray"
subtitle: 'Leetcode978-dynamic programming'
date:       2019-12-03 14:29:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
A subarray ``A[i], A[i+1], ..., A[j]`` of A is said to be turbulent if and only if:
- For ``i <= k < j, A[k] > A[k+1]`` when ``k`` is odd, and ``A[k] < A[k+1]`` when ``k`` is even;
- **OR**, for ``i <= k < j, A[k] > A[k+1]`` when ``k`` is even, and ``A[k] < A[k+1]`` when k is odd.

That is, the subarray is turbulent if the comparison sign flips between each adjacent pair of elements in the subarray.
Return the **length** of a maximum size turbulent subarray of A.
### Example 1:
```c++
Input: [9,4,2,10,7,8,8,1,9]
Output: 5
Explanation: (A[1] > A[2] < A[3] > A[4] < A[5])
```
### Example 2:
```c++
Input: [4,8,12,16]
Output: 2
```
### Example 3:
```c++
Input: [100]
Output: 1
```
### Note:
1. ``1 <= A.length <= 40000``
2. ``0 <= A[i] <= 10^9``

# 解法
## 解法 1
题目给出一个数组，求出其中最长的“躁动”子数组，这里躁动的意思是前后两个值的差值符号不同。  
用动态规划来解，这里为了获得前后差值的符号，新建一个sign数组存储差值符号，差值为正时为1，差值为负时是-1，差值为0就为0。动态数组dp[i]表示到i位置最长躁动子数组的长度，状态转移方程也挺直观，如果当前差值符号为0，表示这个位置的数和前一个位置的数相等，dp[i]置1；如果前一个位置符号为0，表示数组开头元素，dp[i]置2；如果当前差值符号和前一个不同，长度加1；如果当前差值符号和前一个相同，表示上一个子数组已经结束，重新开始新的子数组，dp[i]=2。  
在每个位置用dp[i]和res比较，取大的即可。
```c++
int maxTurbulenceSize(vector<int>& A) 
{
	int res = 1, n = A.size();
	vector<int> sign(n, 0), dp(n, 1);
	for (int i = 1; i < n; ++i)
	{
		if (A[i] - A[i - 1] > 0)
			sign[i] = 1;
		if (A[i] - A[i - 1] < 0)
			sign[i] = - 1;
	}
	for (int i = 1; i < n; ++i)
	{
		if (sign[i] ==0){
			dp[i] = 1;
		}
		else if (sign[i - 1] == 0){
			dp[i] = 2;
		}
		else if (sign[i] * sign[i - 1] == -1){
			dp[i] = dp[i - 1] + 1;
		}
		else {
			dp[i] = 2;
		}
		res = max(res, dp[i]);
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 76.20%   
memory usage: 16.67%  

## 解法 2
和解法1思想一样，只是不新建符号数组sign，直接在A上修改。
```c++
int maxTurbulenceSize(vector<int>& A) 
{
	int res = 1, n = A.size();
	vector<int> sign(n, 0), dp(n, 1);
	int pre = A[0];
	A[0] = 0;
	for (int i = 1; i < n; ++i)
	{
		int diff = A[i] - pre;
		pre = A[i];
		A[i] = diff > 0 ? 1 : (diff < 0 ? -1 : 0);
	}
	for (int i = 1; i < n; ++i)
	{
		if (A[i] * A[i - 1] == -1){
			dp[i] = dp[i - 1] + 1;
		}
		else if (A[i] * A[i - 1] == 1 || (A[i - 1] == 0 && A[i] != 0)){
			dp[i] = 2;
		}
		res = max(res, dp[i]);
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 76.20%   
memory usage: 16.67%  

## 解法 3
继续空间优化，因为dp[i]仅与dp[i-1]有关，用cur存储即可，完全不用新建数组。
```c++
int maxTurbulenceSize(vector<int>& A) 
{
	int res = 1, n = A.size(), cur = 0;
	vector<int> sign(n, 0), dp(n, 1);
	int pre = A[0];
	A[0] = 0;
	for (int i = 1; i < n; ++i)
	{
		int diff = A[i] - pre;
		pre = A[i];
		A[i] = diff > 0 ? 1 : (diff < 0 ? -1 : 0);
	}
	for (int i = 1; i < n; ++i)
	{
		if (A[i] == 0)
			cur = 1;
		else if (A[i - 1] == 0)
			cur = 2;
		else if (A[i] * A[i - 1] == -1)
			++cur;
		else 
			cur = 2;
		res = max(res, cur);
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 92.98%  
memory usage: 16.67%  

# Link
[leetcode-978](https://leetcode.com/problems/longest-turbulent-subarray/)