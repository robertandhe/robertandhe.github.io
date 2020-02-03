---
layout: post
title: "Leetcode123-Best Time to Buy and Sell Stock III"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-16 20:56:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Say you have an array for which the ``ith`` element is the price of a given stock on day i.

Design an algorithm to find the maximum profit. You may complete at most two transactions.
### Note:
You may not engage in multiple transactions at the same time (i.e., you must sell the stock before you buy again).
### Example 1:
```c++
Input: [3,3,5,0,0,3,1,4]
Output: 6
Explanation: Buy on day 4 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.
             Then buy on day 7 (price = 1) and sell on day 8 (price = 4), profit = 4-1 = 3.
```
### Example 2:
```c++
Input: [1,2,3,4,5]
Output: 4
Explanation: Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4.
             Note that you cannot buy on day 1, buy on day 2 and sell them later, as you are
             engaging multiple transactions at the same time. You must sell before buying again.
```
### Example 3:
```c++
Input: [7,6,4,3,1]
Output: 0
Explanation: In this case, no transaction is done, i.e. max profit = 0.
```

# 解法
## 解法 1
这道题我刚开始想的是像Best Time to Buy and Sell Stock II那样求出每笔交易的利润，然后把所有利润排序取至多前两个。这个方法只通过了95%的测试，如{1,2,4,2,5,7,2,4,9,0}无法通过，因为第一次交易在4终止，第二笔交易到7终止，第三笔到9终止，最终利润为{7, 5, 3}，取前两个为12，但因为题目限定了最多两笔交易，第一笔应该在7终止，获得6利润。  
那就只有动态规划了。这里需要两个二维动态数组，一个global[i][j]表示到第i天最多进行j笔交易获得的最大利润，一个local[i][j]表示到第i天最多进行j笔交易并且第j天卖出的交易的最大利润。动态转移方程为
- local[i][j] = max(global[i - 1][j - 1] + max(diff, 0), local[i - 1][j] + diff);其中diff为prices[i] - prices[i - 1],表示相比于前一天价格净增量。local限定了当天卖出，那么只有3种情况：
    1. 当天买，当天卖；净利润增量为0，local[i][j] = global[i - 1][j - 1]
    2. 昨天买，今天卖；净利润增量为diff，local[i][j] = global[i - 1][j - 1] + diff
    3. 昨天之前(包括昨天)买的昨天不卖了，今天卖。相比于昨天，净利润变动为diff，所以local[i][j] = local[i - 1][j] + diff

- global[i][j] = max(global[i - 1][j], local[i][j]);全局最优为在昨天前就进行j笔交易净利润和到今天才进行第j笔交易获得净利润的较大值。

> 这题可以直接扩展到最多k笔交易。

```c++
int maxProfit(vector<int>& prices)
{
	if (prices.empty())
		return 0;
	int n = prices.size();
	vector<vector<int>> global(n, vector<int>(3)), local(n, vector<int>(3));
	for (int i = 1; i < n; ++i)
	{
		int diff = prices[i] - prices[i - 1];
		for (int j = 1; j <= 2; ++j)
		{
			local[i][j] = max(global[i - 1][j - 1] + max(diff, 0), local[i - 1][j] + diff);
			global[i][j] = max(global[i - 1][j], local[i][j]);
		}
	}
	return global[n - 1][2];
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 14.37%  
memory usage: 7.14%  

## 解法 2
空间优化。因为解法1中第i天动态数组值仅与i-1天有关，所以用一维数组就行。
```c++
int maxProfit(vector<int>& prices)
{
	if (prices.empty())
		return 0;
	int n = prices.size();
	vector<int> global(3), local(3);
	for (int i = 1; i < n; ++i)
	{
		int diff = prices[i] - prices[i - 1];
		for (int j = 2; j > 0; --j)
		{
			local[j] = max(global[j - 1] + max(diff, 0), local[j] + diff);
			global[j] = max(global[j], local[j]);
		}
	}
	return global[2];
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 72.01%  
memory usage: 64.29%  

# Link
[leetcode-123](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)  
[solution](https://www.cnblogs.com/grandyang/p/4281975.html)

