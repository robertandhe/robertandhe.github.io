---
layout: post
title: "Leetcode413 - Arithmetic Slices"
subtitle: 'Leetcode-dynamic programming, Math'
date:       2019-11-20 10:41:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - Math
---


# 问题描述
A sequence of number is called arithmetic if it consists of at least three elements and if the difference between any two consecutive elements is the same.  
For example, these are arithmetic sequence:
```c++
1, 3, 5, 7, 9
7, 7, 7, 7
3, -1, -5, -9
```
The following sequence is not arithmetic.
```c++
1, 1, 2, 5, 7
```
A zero-indexed array A consisting of N numbers is given. A slice of that array is any pair of integers (P, Q) such that 0 <= P < Q < N.

A slice (P, Q) of array A is called arithmetic if the sequence:
A[P], A[p + 1], ..., A[Q - 1], A[Q] is arithmetic. In particular, this means that P + 1 < Q.

The function should return the number of arithmetic slices in the array A.
### Example:
```c++
A = [1, 2, 3, 4]

return: 3, for 3 arithmetic slices in A: [1, 2, 3], [2, 3, 4] and [1, 2, 3, 4] itself.
```

# 解法
## 解法 1
无耻暴力解法，对每个起点i，先用diff=A[i+1] - A[i]求出这个序列元素间的差，终点j从i+2到n - 1，在遍历过程中，对每个j，如果A[j] - A[j-1] == diff，res += 1，不满足则break，重置起点i。虽然是暴力解法，但效率还是不错的，因为元素间差值是固定的。
```c++
int numberOfArithmeticSlices(vector<int>& A)
{
	if (A.size() < 3)
		return 0;
	int res = 0, n = A.size();
	for (int i = 0; i <= n - 3; ++i)
	{
		int diff = A[i + 1] - A[i];
		int j = i + 2;
		for (; j < n; ++j)
		{
			if (A[j] - A[j - 1] != diff)
				break;
			++res;
		}
	}
	return res;
}
```
time complexity: O(n^2)  
space complexity: O(1)  
runtime: 63.19%  
memory usage: 100.00%  


## 解法 2
动态规划解法。动态数组dp[i]表示到i位置序列的最长长度，初始化为1，同时用diff存储上个元素间差值。  
在遍历中，如果A[i] - A[i - 1] == diff，表示能够和前面序列组成更长序列，序列长度加1，dp[i] = dp[i - 1] + 1,这就是状态转移方程，那么
res如何加？假设现在的最长序列为1，2，3，4，能够组成3个序列(1, 2, 3)、(2, 3, 4)、(1, 2, 3, 4)。现在来了个5，最长序列成为1，2，3，4，5,新增序列为(3, 4, 5)、(2, 3, 4, 5)、(1, 2, 3, 4, 5)，之前已经考虑过的序列这里不应该再加了，可以看出新增序列个数为序列现在最长长度减2，即res += dp[i] - 2。如果A[i] - A[i - 1] != diff，则要重置序列起点为i - 1, dp[i - 1] = 1, dp[i] = 2。
```c++
int numberOfArithmeticSlices(vector<int>& A)
{
	if (A.size() < 3)
		return 0;
	int res = 0, n = A.size();
	vector<int> dp(n, 1);
	int diff = A[1] - A[0];
	for (int i = 1; i < n; ++i)
	{
		if (A[i] - A[i - 1] == diff){
			dp[i] = dp[i - 1] + 1;
			if(dp[i] > 2)
				res += dp[i] - 2;
		}
		else {
			dp[i - 1] = 1;
			dp[i] = 2;
		}
		diff = A[i] - A[i - 1];
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 63.19%  
memory usage: 93.75%  

## 解法 3
这题有更简单的数学解法，分别求出每个序列的最长长度interval, 这个interval能够组成序列个数为(interval - 2) * (interval - 1) / 2。  
举例来说interval == 5, sequens = {1, 2, 3, 4, 5},各个序列为  
{1, 2, 3}、{1, 2, 3, 4}、{1, 2, 3, 4, 5}  
{2, 3, 4}、{2, 3, 4, 5}  
{3, 4, 5}  
可以看出序列个数为(interval - 2) + (interval - 3) + ... + 1 = (interval - 2) * (interval - 1) / 2。  
```c++
int numberOfArithmeticSlices(vector<int>& A)
{
	int res = 0;
	int left = 0, right = 0, i = 0;
	while (i + 2 < A.size()){
		left = i;
		right = i;
		while (i + 2 < A.size() && (A[i + 2] - A[i + 1] == A[i + 1] - A[i])){
			right = i + 2;
			++i;
		}
		int interval = right - left + 1;
		res += (interval - 2) * (interval - 1) / 2;
		if (left == right)
			++i;
		else 
			i = right;
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 100.00%  
memory usage: 81.25%  

# Link
[leetcode-413](https://leetcode.com/problems/arithmetic-slices/)