---
layout: post
title: "Leetcode322-Coin Change"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-11 22:36:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - recursion
---

# 问题描述
You are given coins of different denominations and a total amount of money amount. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return ``-1``.
### Example 1：
```c++
Input: coins = [1, 2, 5], amount = 11
Output: 3 
Explanation: 11 = 5 + 5 + 1
```
### Example 2：
```c++
Input: coins = [2], amount = 3
Output: -1
```
### Note:
You may assume that you have an infinite number of each kind of coin.

# 解法 
## 解法 1
这道题需要记住中间的状态，很明显是动态规划。设置数组长度为amount+1，初始化为0，0表示对应的钱的量没有出现，dp[i]表示coin多少。需要初始化dp[amount]=1表示amount量已经出现。遍历中需要dp[j]>0表示j钱量已经出现且j>=coins[i]才能找零。当遍历到coin[i]时，状态转移方程需要分情况讨论：
- dp[j-coins[i]]>0,表示j-coins[i]对应的钱的多少已经出现过，那么dp[j - coins[i]] = min(dp[j - coins[i]], dp[j] + 1)
- 否则dp[j - coins[i]] += dp[j] + 1

这里还有一个边界情况就是amout钱的开始，因为要表示amount已经出现，所以dp[amount]=1，但更新时只能dp[amount-coins[i]]=1。
```c++
int coinChange(vector<int>& coins, int amount)
{
	if (amount == 0)
		return 0;
	int n = coins.size();
	sort(coins.begin(), coins.end());
	vector<int> dp(amount + 1, 0);
	dp[amount] = 1;
	for (int i = n - 1; i >= 0; --i)
	{
		if (amount >= coins[i]){
			if (dp[amount - coins[i]] > 0)
				dp[amount - coins[i]] = 1;	
			else 
				dp[amount - coins[i]] += 1;
			if (amount == coins[i])
				return 1;
		}
		for (int j = amount - 1; j >= 0; --j)
		{
			if (dp[j] > 0 && j >= coins[i]){
				if (dp[j - coins[i]] > 0)
					dp[j - coins[i]] = min(dp[j - coins[i]], dp[j] + 1);
				else 
					dp[j - coins[i]] += dp[j] + 1;
			}
		}
	}
	return dp[0] != 0 ? dp[0] : -1;
}
```
time complexity: O(m*n)  
space complexity: O(n)  
runtime: 78.30%  
memory usage: 88.24%  

## 解法 2
上述解法略显繁琐，简化一下，动态数组初始化为-1，dp[amount]=0。转移方程中如果dp[j-coins[i]]!=-1,表示已经出现过，dp[j - coins[i]] = min(dp[j - coins[i]], dp[j] + 1)，否则应该先把dp[j-coins[i]]=0,再加dp[j]+1。
```c++
int coinChange(vector<int>& coins, int amount)
{
	if (amount == 0)
		return 0;
	int n = coins.size();
	sort(coins.begin(), coins.end());
	vector<int> dp(amount + 1, -1);
	dp[amount] = 0;
	for (int i = n - 1; i >= 0; --i)
	{
		for (int j = amount; j >= 0; --j)
		{
			if (dp[j] != -1 && j >= coins[i]){
				if (dp[j - coins[i]] != -1)
					dp[j - coins[i]] = min(dp[j - coins[i]], dp[j] + 1);
				else {
					dp[j - coins[i]] = 0;
					dp[j - coins[i]] += dp[j] + 1;
				}
					
			}
		}
	}
	return dp[0] != -1 ? dp[0] : -1;
}
```
time complexity: O(m*n)  
space complexity: O(n)  
runtime: 78.30%  
memory usage: 84.31%  

## 解法 3
解法1和2都是先遍历coins，再遍历资金。解法3先遍历资金，再遍历coins。而且初始化为amount+1,因为面值为1的coin肯定有，amount资金最多需要amount。状态转移方程为dp[i] = min(dp[i], dp[i - coins[j]] + 1)，这里挺精妙，就算dp[i-coins[j]]没有出现过，dp[i]还是为amount+1，没有更新。最后根据dp[amount]是否为amount+1就能知道能否满足条件。
```c++
int coinChange(vector<int>& coins, int amount)
{
	int n = coins.size();
	sort(coins.begin(), coins.end());
	vector<int> dp(amount + 1, amount + 1);
	dp[0] = 0;
	for (int i = 0; i <= amount; ++i)
	{
		for (int j = 0; j < n; ++j)
		{
			if (i >= coins[j]){
				dp[i] = min(dp[i], dp[i - coins[j]] + 1);
			}
		}
	}
	return dp[amount] != (amount + 1) ? dp[amount] : -1;
}
```
time complexity: O(n * m)  
space complexity: O(n)  
runtime: 34.11%  
memory usage: 86.27%   

## 解法 4
动态规划的递归加记忆数组解法。
```c++
int coinChangeDFS(vector<int>& coins, int target, vector<int>& memo)
{
	if (target < 0)
		return -1;
	if (memo[target] != INT_MAX)
		return memo[target];
	for (int i = 0; i < coins.size(); ++i)
	{
		int tmp = coinChangeDFS(coins, target - coins[i], memo);
		if (tmp >= 0)
			memo[target] = min(memo[target], tmp + 1);
	}
	return memo[target] != INT_MAX ? memo[target] : -1;
}
int coinChange(vector<int>& coins, int amount)
{
	vector<int> memo(amount + 1, INT_MAX);
	memo[0] = 0;
	return coinChangeDFS(coins, amount, memo);
}
```

## 解法 5
动态规划的递归加记忆数组解法，用哈希表来记忆。
```c++
int coinChangeDFS5(vector<int>& coins, int target, unordered_map<int, int>& m)
{
	if (target < 0)
		return -1;
	if (m.count(target))
		return m[target];
	int cur = INT_MAX;
	for (int i = 0; i < coins.size(); ++i)
	{
		int tmp = coinChangeDFS5(coins, target - coins[i], m);
		if (tmp >= 0)
			cur = min(cur, tmp + 1);
	}
	return m[target] = (cur == INT_MAX) ? -1 : cur;
}
int coinChange(vector<int>& coins, int amount)
{
	unordered_map<int, int> m;
	m[0] = 0;
	return coinChangeDFS5(coins, amount, m);
}
```
runtime: 6.03%  
memory usage: 15.69%  

## 解法 6
还是递归解法，不过是根据target能否整除coins[start]来判断。
```c++
void helper(vector<int>& coins, int start, int target, int cur, int& res)
{
	if (start < 0)
		return;
	if (target % coins[start] == 0){
		res = min(res, cur + target / coins[start]);
		return;
	}
	for (int i = target / coins[start]; i >= 0; --i)
	{
		if (cur + i >= res - 1)
			break;
		helper(coins, start - 1, target - i * coins[start], cur + i, res);
	}
}
int coinChange(vector<int>& coins, int amount)
{
	int res = INT_MAX, n = coins.size();
	sort(coins.begin(), coins.end());
	helper(coins, n - 1, amount, 0, res);
	return res == INT_MAX ? -1 : res;
}
```
runtime: 99.81%  
memory usage: 98.04%  

# Link
[leetcode-322](https://leetcode.com/problems/coin-change/)   
[solution3-6](https://www.cnblogs.com/grandyang/p/5138186.html)   