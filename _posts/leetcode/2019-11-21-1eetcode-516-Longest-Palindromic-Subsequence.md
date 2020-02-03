---
layout: post
title: "Leetcode516-Longest Palindromic Subsequence"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-21 11:07:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given a string s, find the longest palindromic subsequence's length in s. You may assume that the maximum length of s is 1000.
### Example 1:
```c++
Input:
"bbbab"
Output:
4
One possible longest palindromic subsequence is "bbbb".
```
### Example 2:
```c++
Input:
"cbbd"
Output:
2
One possible longest palindromic subsequence is "bb".
```

# 解法
## 解法 1
这题适合用动态规划解法，动态数组定义为dp[i][j]为[i, j]范围内最长回文字串的长。i从n-1到0， j从i + 1到n-1。状态转移方程要分两种情况：
1. if (s[i] == s[j]) dp[i][j] = dp[i + 1][j - 1] + 2;
2. else dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]).

当s[i]==s[j]时，这两个字符能补到[i+1, j-1]范围内最长回文字符串的两边是它的长度增加2，所以dp[i][j] = dp[i + 1][j - 1] + 2。当s[i]和s[j]不等时，这两个字符最多只有一个能加到[i+1, j-1]范围内最长回文字符串的某一边，结果取[i, j-1]和[i+1, j]间较大值，这两个值都在之前遍历中已经计算过。  
代码中dp[i][i]初始化为1，因为至少为1。最终返回dp[0][n-1]就是原始字符串中最长回文字符串的长度。
```c++
int longestPalindromeSubseq(string s)
{
	int n = s.size();
	vector<vector<int>> dp(n, vector<int>(n));
	for (int i = n - 1; i >= 0; --i)
	{
		dp[i][i] = 1;
		for (int j = i + 1; j < n; ++j)
		{	
			if (s[i] == s[j])
				dp[i][j] = dp[i + 1][j - 1] + 2;
			else {
				dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
			}
		}
	}
	return dp[0][n - 1];
}
```
time complexity: O(n^2)  
space complexity: O(n^2)  
runtime: 74.79%   
memory usage: 100.00%  


## 解法 2
带记忆数组的递归解法, 和解法1思想一样。
```c++
int helper(string& s, int i, int j, vector<vector<int>>& m)
{
	if (i > j)
		return 0;
	if (i == j)
		return 1;
	if (m[i][j] != -1)
		return m[i][j];
	if (s[i] == s[j]){
		m[i][j] = helper(s, i + 1, j - 1, m) + 2;
	}
	else {
		m[i][j] = max(helper(s, i + 1, j, m), helper(s, i, j - 1, m));
	}
	return m[i][j];
}

int longestPalindromeSubseq3(string s)
{
	int n = s.size();
	vector<vector<int>> memo(n, vector<int>(n, -1));
	return helper(s, 0, n - 1, memo);
}
```
runtime: 91.88%     
memory usage: 10.00%   

# Link
[leetcode-516](https://leetcode.com/problems/longest-palindromic-subsequence/)  
[solution](https://www.cnblogs.com/grandyang/p/6493182.html)
