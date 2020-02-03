---
layout: post
title: "Leetcode983-Minimum Cost For Tickets"
subtitle: 'Leetcode983-dynamic programming'
date:       2019-12-03 20:41:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
In a country popular for train travel, you have planned some train travelling one year in advance.  The days of the year that you will travel is given as an array ``days``.  Each day is an integer from ``1`` to ``365``.

Train tickets are sold in 3 different ways:
- a 1-day pass is sold for ``costs[0]`` dollars;
- a 7-day pass is sold for ``costs[1]`` dollars;
- a 30-day pass is sold for ``costs[2]`` dollars.

The passes allow that many days of consecutive travel.  For example, if we get a 7-day pass on day 2, then we can travel for 7 days: day 2, 3, 4, 5, 6, 7, and 8.

Return the minimum number of dollars you need to travel every day in the given list of ``days``.

### Example 1:
```c++
Input: days = [1,4,6,7,8,20], costs = [2,7,15]
Output: 11
Explanation: 
For example, here is one way to buy passes that lets you travel your travel plan:
On day 1, you bought a 1-day pass for costs[0] = $2, which covered day 1.
On day 3, you bought a 7-day pass for costs[1] = $7, which covered days 3, 4, ..., 9.
On day 20, you bought a 1-day pass for costs[0] = $2, which covered day 20.
In total you spent $11 and covered all the days of your travel.
```
### Example 2:
```c++
Input: days = [1,2,3,4,5,6,7,8,9,10,30,31], costs = [2,7,15]
Output: 17
Explanation: 
For example, here is one way to buy passes that lets you travel your travel plan:
On day 1, you bought a 30-day pass for costs[2] = $15 which covered days 1, 2, ..., 30.
On day 31, you bought a 1-day pass for costs[0] = $2 which covered day 31.
In total you spent $17 and covered all the days of your travel.
```
### Note:
1. ``1 <= days.length <= 365``
2. ``1 <= days[i] <= 365``
3. ``days`` is in strictly increasing order.
4. ``costs.length == 3``
5. ``1 <= costs[i] <= 1000``

# 解法
&emsp;这题给出一个人的旅行计划，在一年内的某些天要外出旅游，具体的天数在days数组里面，同时用联程票可以买，在costs数组中存储联程票的价格，costs[0]为单日票的价格，costs[1]为7日票的价格，costs[2]为月票30日的价格，问最低的旅行费用为多少？  
&emsp;又是最值问题，只有上动态规划了，动态数组dp[i]表示到days数组第i个位置天数最低的旅行费用。最重要的状态转移方程如何求：
- 假设在当天是用的单日票，那么到当前旅行日的费用就等于到上一个旅行日的费用加上单日票的价格，one = costs[0] + dp[i - 1];
- 假设在当天是用的7日票，那么到当前旅行日的费用就等于到买7日旅行票之前的费用加上7日票的价格。j初始化为i - 1,一直往回退，看退到哪里不能用7日票覆盖，这里注意days[i] - days[j] <= 6才是在7日票覆盖范围内，seven = costs[1] + dp[j];
- 假设在当天是用的30日票，那么到当前旅行日的费用就等于到买30日旅行票之前的费用加上30日票的价格。j初始化为i - 1,一直往回退，看退到哪里不能用30日票覆盖，这里注意days[i] - days[j] <= 29才是在7日票覆盖范围内，thirty = costs[1] + dp[j];
- 最终在one, seven, thirty中取最小值就是到当前旅行日的最低费用。

&emsp;这里为了方便计算第一个旅行日的费用，把动态数组大小设置为n+1, dp[0]=0，相应的状态转移方程做出变化即可。最终返回dp.back()。

```c++
int mincostTickets(vector<int>& days, vector<int>& costs) 
{
	int n = days.size();
	vector<int> dp(n + 1, 0);
	for (int i = 1; i <= n; ++i)
	{
		int one = costs[0], seven = costs[1], thirty = costs[2];
		one += dp[i - 1];
		int j = i - 2;
		while (j >= 0 && days[i - 1] - days[j] <= 6)
			--j;
		seven += dp[j + 1];
		j = i - 2;
		while (j >= 0 && days[i - 1] - days[j] <= 29)
			--j;
		thirty += dp[j + 1];
		dp[i] = min(one, min(seven, thirty));	
	}
	return dp.back();
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 77.12%  
memory usage: 75.00%  

# Link
[leetcode-983](https://leetcode.com/problems/minimum-cost-for-tickets/)