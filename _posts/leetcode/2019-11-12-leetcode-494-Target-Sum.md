---
layout: post
title: "Leetcode494-Target Sum"
subtitle: 'Leetcode-dynamic programming, recursion'
date:       2019-11-12 22:35:00
author: "mudux"
header-style: text
catalog: true
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - recursion
---

# 问题描述
You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.

Find out how many ways to assign symbols to make sum of integers equal to target S.
### Example 1:
```c++
Input: nums is [1, 1, 1, 1, 1], S is 3. 
Output: 5
Explanation: 

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

There are 5 ways to assign symbols to make the sum of nums be target 3.
```
### Note:
1. The length of the given array is positive and will not exceed 20.
2. The sum of elements in the given array will not exceed 1000.
3. Your output answer is guaranteed to be fitted in a 32-bit integer.

# 解法
## 解法 1
这种分很多情形的问题无非就是递归和动态规划。先用递归。遍历到数组末尾时如果val==0则满足条件，res+1。
```c++
void helper(vector<int>& nums, int position, long long val, int& res)
{
	if (position == nums.size() && val == 0){
		++res;
		return;
	}
	else if (position == nums.size())
		return;
	helper(nums, position + 1, val + nums[position], res);
	helper(nums, position + 1, val - nums[position], res);
}
int findTargetSumWays(vector<int>& nums, int S)
{
	int res = 0;
	helper(nums, 0, S, res);
	return res;
}
```
runtime: 11.94%  
memory usage: 100.00%  

## 解法 2
还是递归，不过用了记忆数组减少重复计算。
```c++
int helper2(vector<int>& nums, int start, long long val, vector<unordered_map<int, long long>>& m)
{
	if (start == nums.size())
		return val == 0;
	if (m[start].count(val))
		return m[start][val];
	int cnt1 = helper2(nums, start + 1, val + nums[start], m);
	int cnt2 = helper2(nums, start + 1, val - nums[start], m);
	return m[start][val] = cnt1 + cnt2;
}

nt findTargetSumWays(vector<int>& nums, int S)
{
	int n = nums.size();
	vector<unordered_map<int, long long>> m(n);
	return helper2(nums, 0, S, m);
}
```
runtime: 53.17%  
memory usage: 7.69%  
## 解法 3
动态规划。动态数组表示到当前数组位置各个和的个数，vector<unordered_map<int, int>> dp(n + 1)。
```c++
int findTargetSumWays(vector<int>& nums, int S)
{
	int n = nums.size();
	vector<unordered_map<int, int>> dp(n + 1);
	dp[0][0] = 1;
	for (int i = 0; i < n; ++i)
	{
		for (auto& a : dp[i])
		{
			dp[i + 1][a.first + nums[i]] += a.second;
			dp[i + 1][a.first - nums[i]] += a.second;
		}
	}
	return dp[n][S];
}
```
runtime: 59.40%  
memory usage: 15.38%  

## 解法 4
和解法3一样，因为当前位置i更新是仅和上一位置i-1有关，但更新过程中会改变哈希表，所以建立临时哈希表，更新完放回。
```c++
int findTargetSumWays(vector<int>& nums, int S)
{
	int n = nums.size();
	unordered_map<int, int> dp;
	dp[0] = 1;
	for (int i = 0; i < n; ++i)
	{
		unordered_map<int, int> t;
		for (auto& a : dp)
		{
			t[a.first + nums[i]] += a.second;
			t[a.first - nums[i]] += a.second;
		}
		dp = t;
	}
	return dp[S];
}
```
runtime: 56.54%  
memory usage: 7.69%  

# Link
[leetcode-494](https://leetcode.com/problems/target-sum/)  
[solution 2-4](https://www.cnblogs.com/grandyang/p/6395843.html?utm_source=itdadao&utm_medium=referral)  