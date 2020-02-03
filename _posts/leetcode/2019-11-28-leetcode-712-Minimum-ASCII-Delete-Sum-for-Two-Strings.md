---
layout: post
title: "Leetcode712-Minimum ASCII Delete Sum for Two Strings"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-28 15:50:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given two strings ``s1``, ``s2``, find the lowest ASCII sum of deleted characters to make two strings equal.
### Example 1:
```c++
Input: s1 = "sea", s2 = "eat"
Output: 231
Explanation: Deleting "s" from "sea" adds the ASCII value of "s" (115) to the sum.
Deleting "t" from "eat" adds 116 to the sum.
At the end, both strings are equal, and 115 + 116 = 231 is the minimum sum possible to achieve this.
```
### Example 2:
```c++
Input: s1 = "delete", s2 = "leet"
Output: 403
Explanation: Deleting "dee" from "delete" to turn the string into "let",
adds 100[d]+101[e]+101[e] to the sum.  Deleting "e" from "leet" adds 101[e] to the sum.
At the end, both strings are equal to "let", and the answer is 100+101+101+101 = 403.
If instead we turned both strings into "lee" or "eet", we would get answers of 433 or 417, which are higher.
```
### Note:
- 0 < s1.length, s2.length <= 1000.
- All elements of each string will have an ASCII value in [``97, 122``].

# 解法
## 解法 1
&emsp;这道题和之前那道[583. Delete Operation for Two Strings](https://leetcode.com/problems/delete-operation-for-two-strings/)很相似。这里是给出两个字符串，求出使它们相等需要删除字符的最小ASCII的和。  
&emsp;还是动态规划来解，dp[i][j]表示使s1前i个字符和s2前j个字符相等需要删除字符的最小ASCII码和。dp[0][j]和dp[i][0]边界情况就按照s2所有字符相加和s1所有字符相加即可。状态转移方程为：
- 当s1[i - 1] == s2[j - 1],那么需要删除的字符还是和之前相同，dp[i][j] = dp[i - 1][j - 1];
- 当s1[i - 1]和s2[j - 1]不等时，需要删除其中一个，当然是哪个小删哪个了，dp[i][j] = min(dp[i][j - 1] + s2[j - 1], dp[i - 1][j] + s1[i - 1]).

&emsp;最终返回dp[m][n]。
```c++
int minimumDeleteSum(string s1, string s2)
{
	int m = s1.size(), n = s2.size();
	vector<vector<int>> dp(m + 1, vector<int>(n + 1));
	for (int j = 1; j <= n; ++j)
		dp[0][j] = dp[0][j - 1] + s2[j - 1];
	for (int i = 1; i <= m; ++i)
	{
		dp[i][0] = dp[i - 1][0] + s1[i - 1];
		for (int j = 1; j <= n; ++j)
		{
			if (s1[i - 1] == s2[j - 1])
				dp[i][j] = dp[i - 1][j - 1];
			else {
				dp[i][j] = min(dp[i - 1][j] + s1[i - 1], dp[i][j - 1] + s2[j - 1]);
			}
		}
	}
	return dp[m][n];
}
```
time complexity: O(m * n)  
space complexity: O(m * n)  
runtime: 80.20%  
memory usage: 77.78%  

## 解法 2
&emsp;和解法1思想相反，这里求他们相同的最大子串。dp[i][j]表示s1前i个字符和s2前j个字符相同子串的最大ASCII值。当s1[i - 1] == s2[j - 1]时，相同子串需要加这个字符，dp[i][j] = dp[i - 1][j - 1] + s1[i - 1];当s1[i - ]和s2[j - 1]不等时，尽量使他们子串大即可，dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])。

&emsp;最终dp[m][n]是最大子串ASCII码值，两个字符串ASCII码值之和减去2倍最大子串就是返回值。
```c++
int minimumDeleteSum(string s1, string s2)
{
	int m = s1.size(), n = s2.size();
	vector<vector<int>> dp(m + 1, vector<int>(n + 1));
	for (int i = 1; i <= m; ++i)
	{
		for (int j = 1; j <= n; ++j)
		{
			if (s1[i - 1] == s2[j - 1])
				dp[i][j] = dp[i - 1][j - 1] + s1[i - 1];
			else 
				dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
		}
	}
	int t1 = accumulate(s1.begin(), s1.end(), 0);
	int t2 = accumulate(s2.begin(), s2.end(), 0);
	return t1 + t2 - 2 * dp[m][n];
}
```
time complexity: O(m * n)  
space complexity: O(m * n)  
runtime: 93.18%  
memory usage: 44.44%  

# Link
[leetcode-712](https://leetcode.com/problems/minimum-ascii-delete-sum-for-two-strings/)