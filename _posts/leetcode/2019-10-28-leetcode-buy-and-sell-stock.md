---
layout: post
title: "Leetcode-buy and sell stock"
subtitle: 'Leetcode-stack'
date:       2019-10-28 10:34:00
author: "mudux"
header-style: text
catalog: true
tags:
  - algorithm
  - leetcode
  - greedy
  - dynamic programming
---

在这个笔记总结一下leetcode上出现的几个和stock有关的题目。
## [121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)
### 分析
这道题限制一次交易，那么直接用最大值减最小值即可。
### 解法
```c++
int maxProfit(vector<int>& prices) {
    if (prices.empty())
        return 0;
    int res = 0, n = prices.size(), mn = prices[0];
    for (int i = 1; i < n; ++i)
    {
        mn = min(mn, prices[i]);
        res = max(res, prices[i] - mn);
    }
    return res;     
}
```

## [122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)
### 分析
这道题不限制交易次数，属于贪心问题。有利润就收。当前价格小于明天价格，就可今天买入，明天卖出。
### 解法
```c++
int maxProfit(vector<int>& prices)
{
    if (prices.empty())
        return 0;
    int res = 0;
    for (int i = 0; i < prices.size() - 1; ++i)
    {
        if (prices[i] < prices[i + 1])
            res += prices[i + 1] - prices[i];
    }
    return res;
}
```

## [714. Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)
### 分析
这道题不限制交易次数，但是有交易费用，只有卖出价-买入价大于交易费用才有利润可赚。
### 解法1
用动态规划解，用两个向量sold和hold表示不持有和持有的利润。第i天不持有利润为i-1天不持有利润和i-1天持有今天卖出的最大值，即``sold[i] = max(sold[i - 1], hold[i - 1] + prices[i] - fee)``。第i天持有利润为i-1天持有和i-1天不持有今天买入的利润最大值，即``hold[i] = max(hold[i - 1], sold[i - 1] - prices[i])``。
```c++
int maxProfit(vector<int>& prices, int fee)
{
	vector<int> sold(prices.size(), 0), hold = sold;
	hold[0] = -prices[0];
	for (int i = 1; i < prices.size(); ++i)
	{
		sold[i] = max(sold[i - 1], hold[i - 1] + prices[i] - fee);
		hold[i] = max(hold[i - 1], sold[i - 1] - prices[i]);
	}
	return sold.back();
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 88.32%  
memory usage: 11.76%  
### 解法2
在解法1中，动态规划仅和上一天有关，所以可用单变量保存中间结果。sold更新时要用中间变量过渡。
```c++
int maxProfit(vector<int>& prices, int fee)
{
	int sold = 0, hold = -prices[0];
	for (int& price: prices)
	{
		int t = sold;
		sold = max(sold, hold + price - fee);
		hold = max(hold, t - price);
	}
	return sold;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 88.32%  
memory usage: 94.12%  


## 未完待续