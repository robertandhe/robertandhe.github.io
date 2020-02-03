---
layout: post
title: "Leetcode221-Maximum Square"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-15 22:45:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given a 2D binary matrix filled with 0's and 1's, find the largest square containing only 1's and return its area.
### Example:
```c++
Input: 

1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0

Output: 4
```

# 解法
## 解法 1
先上暴力解法:)。简单来说就是把每个位置当作正方形右下角位置，然后往左上方遍历。遍历范围为当前位置(i, j)的较小值范围，如果在遍历途中遇到非1的点，说明无法组成正方形了，后面就都不用遍历了。
```c++
int maximalSquare(vector<vector<char>>& matrix)
{
	if (matrix.empty())
		return 0;
	int res = 0, m = matrix.size(), n = matrix[0].size();
	for (int i = 0; i < m; ++i)
	{
		for (int j = 0; j < n; ++j)
		{
			if (matrix[i][j] == '0')
				continue;
			int k = min(i, j);
			int flag = 1;
			for (int t = 0; t <= k && flag; ++t)
			{
				for (int r = i - t; r <= i && flag; ++r)
				{
					for (int c = j - t; c <= j && flag; ++c)
					{
						if (matrix[r][c] != '1')
							flag = 0;
					}
				}
				if (flag)
					res = max(res, (t + 1) * (t + 1));
			}
		}
	}
	return res;
}
```
ime complexity: O(m^2 * n ^ 2)  
space complexity: O(1)  
runtime: 36.56%  
memory usage: 100.00%  

## 解法 2
区间和解法。建立和原始matrix等size的区间和数组，区间和数组每个位置数值表示当前位置左方，上方，左上方所有位置1的个数。然后对每一个位置，
按照(i, j)的较小值遍历区间，如果区间内1的个数等于边长的平方说明可以组成正方形，更新res。
```c++
int maximalSquare(vector<vector<char>>& matrix)
{
	if (matrix.empty() || matrix[0].empty())
		return 0;
	int res = 0, m = matrix.size(), n = matrix[0].size();
	vector<vector<int>> sums(m, vector<int>(n));
	for (int i = 0; i < m; ++i)
	{
		for (int j = 0; j < n; ++j)
		{
			int t = matrix[i][j] - '0';
			if ( i > 0)
				t += sums[i - 1][j];
			if (j > 0 )
				t += sums[i][j - 1];
			if (i > 0 && j > 0)
				t -= sums[i - 1][j - 1];
			sums[i][j] = t;
			int K = min(i, j);
			int cnt = 1;
			for (int k = K; k >= 0; --k)
			{
				int s = sums[i][j];
				if (i - cnt >= 0)
					s -= sums[i - cnt][j];
				if (j - cnt >= 0)
					s -= sums[i][j - cnt];
				if (i - cnt >= 0 && j - cnt >= 0)
					s += sums[i - cnt][j - cnt];
				if (s == cnt * cnt)
					res = max(res, s);
				++cnt;
			}
		}
	}
	return res;
}
```
time complexity: O(m * n * min(m, n))  
space complexity: O(m * n)  
runtime: 5.47%  
memory usage: 51.85%  

## 解法 3
动态规划解法。二维动态数组表示以(i, j)位置为正方形右下角所能组成正方形的最大边长。那么状态转移方程为
- 当前位置为'0'，无法组成正方形了，dp[i][j] = 0;
- 当前位置为'1',最大边长为左方，上方，左上方最大边长的最小值加1.
遍历过程中不停更新res即可。这里还需处理边界情况，那就是第一行或者第一列中根据matrix[i][j]为'0'或者'1'更新dp[i][j]。
```c++
int maximalSquare2(vector<vector<char>>& matrix)
{
	if (matrix.empty())
		return 0;
	int res = 0, m = matrix.size(), n = matrix[0].size();
	vector<vector<int>> dp(m, vector<int>(n));
	for (int i = 0; i < m; ++i)
	{
		for (int j = 0; j < n; ++j)
		{
			if (i == 0 || j == 0)
				dp[i][j] = matrix[i][j] - '0';
			else {
				if (matrix[i][j] == '0')
					dp[i][j] = 0;
				else 
					dp[i][j] = min(dp[i - 1][j - 1], min(dp[i - 1][j], dp[i][j - 1])) + 1;
			}
			res = max(res, dp[i][j]);
		}
	}
	return res * res;
}
```
time complexity: O(m * n)  
space complexity: O(m * n)  
runtime: 19.16%  
memory usage: 62.96%  

## 解法 4
因为解法3动态规划是仅与三个位置有关：左方，上方，左上方。那么使用一个一维动态数组即可。这里按照行优先遍历，dp长度设置为m+1。遍历中pre表示左上方最长边长，dp[i - 1]就表示上方最长边长，dp[i]表示左方最长边长。其余思想与解法3一样。
```c++
int maximalSquare3(vector<vector<char>>& matrix)
{
	if (matrix.empty())
		return 0;
	int res = 0, m = matrix.size(), n = matrix[0].size(), pre = 0;
	vector<int> dp(m + 1, 0);
	for (int j = 0; j < n; ++j)
	{
		for (int i = 1; i <= m; ++i)
		{
			if (matrix[i - 1][j] == '0')
				dp[i] = 0;
			else {
				int t = dp[i];
				dp[i] = min(dp[i], min(dp[i - 1], pre)) + 1;
				pre = t;
			}
			res = max(res, dp[i]);
		}
	}
	return res * res;
}
```
time complexity: O(m * n)  
space complexity: O(m)  
runtime: 78.53%  
memory usage: 100.00%  

# Link
[leetcode-221](https://leetcode.com/problems/maximal-square/)  
[solution 2, 4](https://www.cnblogs.com/grandyang/p/4550604.html)  