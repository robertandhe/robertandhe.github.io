---
layout: post
title: "Leetcode474-Ones and Zeroes"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-13 10:56:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
In the computer world, use restricted resource you have to generate maximum benefit is what we always want to pursue.

For now, suppose you are a dominator of ``m`` 0s and ``n`` 1s respectively. On the other hand, there is an array with strings consisting of only ``0s`` and ``1s``.

Now your task is to find the maximum number of strings that you can form with given ``m 0s`` and ``n 1s``. Each 0 and 1 can be used at most once.
### Note:
1. The given numbers of 0s and 1s will both not exceed 100
2. The size of given string array won't exceed 600.]

### Example 1:
```c++
Input: Array = {"10", "0001", "111001", "1", "0"}, m = 5, n = 3
Output: 4

Explanation: This are totally 4 strings can be formed by the using of 5 0s and 3 1s, which are “10,”0001”,”1”,”0”
```
### Example 2:
```c++
Input: Array = {"10", "0", "1"}, m = 1, n = 1
Output: 2

Explanation: You could form "10", but then you'd have nothing left. Better form "0" and "1".
```

# 解法
这道题一看就是动态规划解法，如果用贪心先给数组按长度排序，可能出现1或0消耗过快问题，所以需要穷尽所有情形，只能用动态规划。
因为用m个0，n个1，那么动态数组肯定是要两维，dp[i][j]表示i个0，j个1所能组成的最大字符串个数。剩下的问题就是求解状态转移方程，
对每个遍历到的字符串均假设它在最终结果里面，统计它0和1的个数表示为zeros和ones，它的位置也不知，只能穷尽所有情形，下标i从m递减
到zeros,下表j从n递减到ones，对中间每个位置dp[i][j] = max(dp[i][j], dp[i - zeros][j - ones] + 1),这就是状态转移方程。
```c++
int findMaxForm(vector<string>& strs, int m, int n)
{
	vector<vector<int>> dp(m + 1, vector<int> (n + 1));
	for (string& str : strs)
	{
		int zeros = 0, ones = 0;
		for (char& c : str)
			c == '0' ? ++zeros : ++ones;
		for (int i = m; i >= zeros; --i)
		{
			for (int j = n; j >= ones; --j)
				dp[i][j] = max(dp[i][j], dp[i - zeros][j - ones] + 1);
		}
	}
	return dp[m][n];
}
```
time complexity: O(n)  
space complexity: O(n^2)  
runtime: 70.76%  
memory usage: 100.00%   

# Link
[leetcode-474](https://leetcode.com/problems/ones-and-zeroes/)  