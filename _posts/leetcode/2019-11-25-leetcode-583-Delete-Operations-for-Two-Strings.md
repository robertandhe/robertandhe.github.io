---
layout: post
title: "Leetcode583-Delete Operation for Two Strings"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-25 17:30:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given two words word1 and word2, find the minimum number of steps required to make word1 and word2 the same, where in each step you can delete one character in either string.
### Example 1:
```c++
Input: "sea", "eat"
Output: 2
Explanation: You need one step to make "sea" to "ea" and another step to make "eat" to "ea".
```
### Note:
1. The length of given words won't exceed 500.
2. Characters in given words can only be lower-case letters.

# 解法
## 解法 1
换个角度，建立动态数组dp[i][j]表示word1[0: i)和word2[0, j)子字符串最长公共子字符串的长度。那么状态转移方程：
- 如果word1[i - 1] == word2[j - 1],表示这个字符是公共字符，dp[i][j] = dp[i - 1][j - 1] + 1;
- 否则这个字符就不是公共字符，但还是有错位公共子字符串存在，dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])。

```c++
int minDistance(string word1, string word2)
{
	int n1 = word1.size(), n2 = word2.size();
	vector<vector<int>> dp(n1 + 1, vector<int>(n2 + 1));
	for (int i = 1; i <= n1; ++i)
	{
		for (int j = 1; j <= n2; ++j)
		{
			if (word1[i - 1] == word2[j - 1])
				dp[i][j] = dp[i - 1][j - 1] + 1;
			else 	
				dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
		}
	}
	return n1 + n2 - 2 * dp[n1][n2];
}
```
time complexity: O(n1 * n2)  
space complexity: O(n1 * n2)  
runtime: 78.66%  
memory usage: 55.56%  

## 解法 2
动态数组dp[i][j]表示使word1[0: i)和word2[0, j)子字符串相等需要做的删除次数。状态转移方程为：
- 如果word1[i - 1] == word2[j - 1],表示这个字符是公共字符，不需要删除，dp[i][j] = dp[i - 1][j - 1];
- 否则这个字符就不是公共字符，但还是有错位公共子字符串存在，dp[i][j] = 1 + min(dp[i - 1][j], dp[i][j - 1])。

```c++
int minDistance2(string word1, string word2)
{
	int n1 = word1.size(), n2 = word2.size();
	vector<vector<int>> dp(n1 + 1, vector<int>(n2 + 1));
	for (int i = 0; i <= n1; ++i)
		dp[i][0] = i;
	for (int j = 0; j <= n2; ++j)
		dp[0][j] = j;
	for (int i = 1; i <= n1; ++i)
	{
		for (int j = 1; j <= n2; ++j)
		{
			if (word1[i] == word2[j])
				dp[i][j] = dp[i - 1][j - 1];
			else {
				dp[i][j] = 1 + min(dp[i - 1][j], dp[i][j - 1]);
			}
		}
	}
	return dp[n1][n2];
}
```
time complexity: O(n1 * n2)  
space complexity: O(n1 * n2)  
runtime: 30.07%  
memory usage: 55.56%  

# Link
[leetcode-583](https://leetcode.com/problems/delete-operation-for-two-strings/)