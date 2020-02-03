---
layout: post
title: "Leetcode188-Best Time to Buy and Sell Stock IV"
subtitle: 'Leetcode188-dynamic programming'
date:       2019-11-18 17:21:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Say you have an array for which the i-th element is the price of a given stock on day i.

Design an algorithm to find the maximum profit. You may complete at most k transactions.
### Note:
You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).

### Example 1:
```c++
Input: [2,4,1], k = 2
Output: 2
Explanation: Buy on day 1 (price = 2) and sell on day 2 (price = 4), profit = 4-2 = 2.
```
### Example 2:
```c++
Input: [3,2,6,5,0,3], k = 2
Output: 7
Explanation: Buy on day 2 (price = 2) and sell on day 3 (price = 6), profit = 6-2 = 4.
             Then buy on day 5 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.
```

# 解法
## 解法 1
这题和之前这道[Leetcode123-Best Time to Buy and Sell Stock III](https://hexinlin.top/2019/11/16/leetcode-123-Best-TIme-to-Buy-and-Sell-Sock-III/)如出一辙。那里限定最多交易2次，这里是最多交易k次。换汤不换药。  
但这里测试样例里面有个坑，有k大到几十万的测试样例，但k>prices.size()，这时就相当于不限交易次数了，和[122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)解法一样。所以这题是122和123两题的组合，没啥创新。
```c++
int maxProfitHelper(vector<int>& prices)
{
	int res = 0;
	for (int i = 1; i < prices.size(); ++i)
	{
		if (prices[i] > prices[i - 1])
			res += prices[i] - prices[i - 1];
	}
	return res;
}

int maxProfit(int k, vector<int>& prices)
{
	if (prices.empty())
		return 0;
	if (k >= prices.size())
		return maxProfitHelper(prices);
	int n = prices.size();
	vector<vector<int>> global(n, vector<int>(k + 1)), local(n, vector<int>(k + 1));
	for (int i = 1; i < n; ++i)
	{
		int diff = prices[i] - prices[i - 1];
		for (int j = 1; j <= k; ++j)
		{
			local[i][j] = max(global[i - 1][j - 1] + max(diff, 0), local[i - 1][j] + diff);
			global[i][j] = max(global[i - 1][j], local[i][j]);
		}
	}
	return global[n - 1][k];
}
```
time complexity: O(n * k)  
space complexity: O(n * k)  
runtime: 74.10%  
memory usage: 11.11%   

## 解法 2
空间优化。
```c++

int maxProfitHelper2(vector<int>& prices)
{
	int res = 0;
	for (int i = 1; i < prices.size(); ++i)
	{
		if (prices[i] > prices[i - 1])
			res += prices[i] - prices[i - 1];
	}
	return res;
}

int maxProfit2(int k, vector<int>& prices)
{
	if (prices.empty())
		return 0;
	if (k >= prices.size())
		return maxProfitHelper2(prices);		
	int n = prices.size();
	vector<int> global(k + 1), local(k + 1);
	for (int i = 1; i < n; ++i)
	{
		int diff = prices[i] - prices[i - 1];
		for (int j = k; j >= 1; --j)
		{
			local[j] = max(global[j - 1] + max(diff, 0), local[j] + diff);
			global[j] = max(global[j], local[j]);
		}
	}
	return global[k];
}
```
time complexity: O(n * k)  
space complexity: O(k)  
runtime: 74.10%  
memory usage: 50.00%  

# Link
[leetcode-188](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)  
[leetcode-122](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)  
[leetcode-123](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)  
