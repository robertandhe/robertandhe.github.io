---
layout: post
title: "Leetcode309-Best Time to Buy and Sell Stock with Cooldown"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-16 18:26:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Say you have an array for which the ith element is the price of a given stock on day i.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times) with the following restrictions:

You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
After you sell your stock, you cannot buy stock on next day. (ie, cooldown 1 day)

### Example:
```c++
Input: [1,2,3,0,2]
Output: 3 
Explanation: transactions = [buy, sell, cooldown, buy, sell]
```

# 解法
## 解法 1
这题一看就要动态规划，但是没搞出来，看大佬的。在每一天只能处于三种状态：买，卖，休息。![如下图所示](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-16/post.PNG)
所以需要三个状态数组s0, s1, s2，分别表示第i天休息，买，卖的最大利润。接下来就只需
求出状态转移方程：
- s0[i] = max(s0[i - 1], s2[i - 1]);表示休息状态下的利润为继续休息或者前一天卖出的利润
- s1[i] = max(s1[i - 1], s0[i - 1] - prices[i]);表示买入状态下的利润为买入后休息或者前一天买入
- s2[i] = s1[i - 1] + prices[i];表示卖出状态利润为上一个买入状态利润加上今天卖出价格

最终返回s0.back()和s2.back()的最大值即可，因为s1.back()肯定不是最大值。
```c++
int maxProfit(vector<int>& prices)
{
	if (prices.empty())
		return 0;
	int n = prices.size();
	vector<int> s0(n, 0), s1(n, 0), s2(n, 0);
	s0[0] = 0;
	s1[0] = -prices[0];
	s2[0] = INT_MIN;
	for (int i = 1; i < n; ++i)
	{
		s0[i] = max(s0[i - 1], s2[i - 1]);
		s1[i] = max(s1[i - 1], s0[i - 1] - prices[i]);
		s2[i] = s1[i - 1] + prices[i];
	}
	return max(s0.back(), s2.back());
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 74.67%  
memory usage: 55.56%  

## 解法 2
解法1空间优化，因为当前天仅与上一天有关，用单变量替代即可。
```c++
int maxProfit(vector<int>& prices)
{
	if (prices.empty())
		return 0;
	int n = prices.size();
	int s0 = 0, s1 = -prices[0], s2 = INT_MIN, preS0 = 0, preS1 = 0;
	for (int i = 1; i < n; ++i)
	{
		preS0 = s0;
		s0 = max(s0, s2);
		preS1 = s1;
		s1 = max(s1, preS0 - prices[i]);
		s2 = preS1 + prices[i];
	}
	return max(s0, s2);
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 74.67%  
memory usage: 92.59%  

# Link
[leetcode-309](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)  
[solution 1](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/discuss/75928/Share-my-DP-solution-(By-State-Machine-Thinking))