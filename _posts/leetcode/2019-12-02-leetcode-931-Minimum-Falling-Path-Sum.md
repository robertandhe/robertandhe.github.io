---
layout: post
title: "Leetcode931 - Minimum Falling Path Sum"
subtitle: 'Leetcode-dynamic programming, recursion'
date:       2019-12-02 20:25:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - recursion
---

# 问题描述
Given a **square** array of integers ``A``, we want the **minimum** sum of a falling path through ``A``.

A falling path starts at any element in the first row, and chooses one element from each row.  The next row's choice must be in a column that is different from the previous row's column by at most one.
### Example 1:
```c++
Input: [[1,2,3],[4,5,6],[7,8,9]]
Output: 12
Explanation: 
The possible falling paths are:
[1,4,7], [1,4,8], [1,5,7], [1,5,8], [1,5,9]
[2,4,7], [2,4,8], [2,5,7], [2,5,8], [2,5,9], [2,6,8], [2,6,9]
[3,5,7], [3,5,8], [3,5,9], [3,6,8], [3,6,9]
The falling path with the smallest sum is [1,4,7], so the answer is 12.
```
### Note:
1. ``1 <= A.length == A[0].length <= 100``
2. ``-100 <= A[i][j] <= 100``

# 解法
## 解法 1
&emsp;这道题给出一个二维数组，要求找出最小的下降路径，从第一行某个数开始，只能从下一行左边那一列，右边那一列，正下方这列选择下降路径。  
&emsp;这中最值问题一看就是动态规划解法了，动态数组肯定也是二维了，dp[i][j]表示到达(i, j)位置最小的下降路径和，那么状态转移方程如何求，假设当前处于(i, j)位置，怎么到达它，只能从(i - 1, j - 1), (i - 1, j), (i - 1, j + 1)三个位置到达，选择最小那个即可，当然还需考虑边界条件，最左边，最右边需要区分处理，因为下一行只有上一行有关，所以直接在A数组上修改即可，都不用申请状态数组。  
&emsp;最终返回值就是最后一行的最小值。  
```c++
int minFallingPathSum3(vector<vector<int>>& A) 
{
	int res = INT_MAX, n = A.size();
	for (int i = 1; i < n; ++i)
	{
		for (int j = 0; j < n; ++j)
		{
			int a = INT_MAX, b = INT_MAX, c = INT_MAX;
			if (j > 0)
				a = A[i - 1][j - 1];
			b = A[i - 1][j];
			if (j < n - 1)
				c = A[i - 1][j + 1];
			A[i][j] += min(a, min(b, c));
		}
	}
	for (int i = 0; i < n; ++i)
		res = min(res, A[n - 1][i]);
	return res;
}
```
time complexity: O(n^2)  
space complexity: O(1)  
runtime:  99.40%  
memory usage: 100.00%  

## 解法 2
&emsp;能用动态规划绝对也能用递归，为了减少重复计算，这里用带记忆数组得递归，之所以有重复计算，是因为(i, j)可以从(i - 1, j - 1), (i - 1, j), (i - 1,j + 1)到达，无论是从上面哪个位置到达，只要到了(i, j)位置，就是同样处理了。   
&emsp;进入row, col位置时，用res存储当前值，如果还有下一行，从下面(row + 1, col - 1), (row + 1, col), (row + 1， col + 1)三个位置中选最小的值来加，还是需要注意边界条件。用memo[row][col]=res存储中间结果。
```c++
int helper(vector<vector<int>>& A, int row, int col, vector<vector<int>>& memo)
{
	if (memo[row][col] != INT_MAX)
		return memo[row][col];
	int res = A[row][col];
	int a = INT_MAX, b = INT_MAX, c = INT_MAX;
	if (row < A.size() - 1)
		a = helper(A, row + 1, col, memo);
	if(row < A.size() - 1 && col > 0)
		b = helper(A, row + 1, col - 1, memo);
	if (row < A.size() - 1 && col < A.size() - 1)
		c = helper(A, row + 1, col + 1, memo);
	if (row < A.size() - 1)
		res += min(a, min(b, c));
	memo[row][col] = res;
	return res;
}

int minFallingPathSum(vector<vector<int>>& A) 
{
	int res = INT_MAX, n = A.size();
	vector<vector<int>> memo(n, vector<int>(n, INT_MAX));
	for (int i = 0; i < n; ++i)
		res = min(res, helper(A, 0, i, memo));
	return res;
}
```
runtime:  45.33%  
memory usage: 33.33%  

## 解法 3
&emsp;和解法2思想一样，写的美观一点，运行速度快一点。
```c++
int helper(vector<vector<int>>& A, int row, int col, vector<vector<int>>& memo)
{
	if (row == A.size())
		return INT_MAX;
	if (memo[row][col] != INT_MAX)
		return memo[row][col];
	int res = A[row][col];
	int a = INT_MAX, b = INT_MAX, c = INT_MAX;
	a = helper(A, row + 1, col, memo);
	if (col > 0)
		b = helper(A, row + 1, col - 1, memo);
	if (col < A.size() - 1)
		c = helper(A, row + 1, col + 1, memo);
	if (a != INT_MAX || b != INT_MAX || c != INT_MAX)
		res += min(a, min(b, c));
	memo[row][col] = res;
	return res;
}

int minFallingPathSum(vector<vector<int>>& A) 
{
	int res = INT_MAX, n = A.size();
	vector<vector<int>> memo(n, vector<int>(n, INT_MAX));
	for (int i = 0; i < n; ++i)
		res = min(res, helper(A, 0, i, memo));
	return res;
}
```
runtime:  89.01%  
memory usage: 33.33%   

# Link
[leetcode-931](https://leetcode.com/problems/minimum-falling-path-sum/)