---
layout: post
title: "Leetcode1027 - Longest Arithmetic Sequence"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-29 20:46:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given an array ``A`` of integers, return the **length** of the longest arithmetic subsequence in ``A``.

Recall that a subsequence of ``A`` is a list ``A[i_1], A[i_2], ..., A[i_k]`` with ``0 <= i_1 < i_2 < ... < i_k <= A.length - 1``, and that a sequence B is arithmetic if ``B[i+1] - B[i]`` are all the same value (for ``0 <= i < B.length - 1``).
### Example 1:
```c++
Input: [3,6,9,12]
Output: 4
Explanation: 
The whole array is an arithmetic sequence with steps of length = 3.
```
### Example 2:
```c++
Input: [9,4,7,2,10]
Output: 3
Explanation: 
The longest arithmetic subsequence is [4,7,10].
```
### Example 3:
```c++
Input: [20,1,15,3,10,5,8]
Output: 4
Explanation: 
The longest arithmetic subsequence is [20,15,10,5].
```
### Note:
1. ``2 <= A.length <= 2000``
2. ``0 <= A[i] <= 10000``

# 解法
&emsp;这题和之前某道题有点像，不过题号我忘了:)。这题中要给出一个数组，求出中间最长的Arithmetic Sequence，就是前后元素差值相等的序列。  
&emsp;还是动态规划来解，这里需要保存每个位置对应差值的序列长度，一维动态数组肯定是不行的。同时也不知道数组最小值和最大值差的有点远，二维数组过度消耗内存，所以就这能用哈希表动态数组了,dp[i]为unordered_map<int, int>存储i位置各个差值diff对应的序列长度。  
&emsp;状态转移方程也挺简单，对每个i位置，往前遍历即可，假设位置为j，差值diff = A[i] - A[j]。此时差值为diff的序列最小长度肯定是2，如果dp[i].count(diff)为空，则初始化为dp[i][diff] = 2，还需判断在j位置时能否组成差值为diff的序列，如果有dp[i][diff] = max(dp[i][diff], dp[j][diff] + 1)。  
&emsp;在每个位置res都要及时更新，最终返回res。
```c++
int longestArithSeqLength(vector<int>& A) 
{
	int res = 0, n = A.size();
	vector<unordered_map<int, int>> dp(n);
	for (int i = 1; i < n; ++i)
	{
		for (int j = i - 1; j >= 0; --j)
		{
			int diff = A[i] - A[j];
			if (!dp[i].count(diff))
				dp[i][diff] = 2;
			if (dp[j].count(diff)){
				dp[i][diff] = max(dp[i][diff], dp[j][diff] + 1);
			}
			res = max(res, dp[i][diff]);
		}
	}
	return res;
}
```
time complexity: O(n^2)  
space complexity: O(n^2)  
run time: 66.16%  
memory usage: 93.33%  

# Link
[leetcode-1027](https://leetcode.com/problems/longest-arithmetic-sequence/)