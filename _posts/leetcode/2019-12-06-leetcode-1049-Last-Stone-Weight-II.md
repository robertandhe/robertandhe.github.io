---
layout: post
title: "Leetcode1049 - Last Stone Weight II"
subtitle: 'Leetcode1049-dynamic programming'
date:       2019-12-06 10:20:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
We have a collection of rocks, each rock has a positive integer weight.

Each turn, we choose **any two rocks** and smash them together.  Suppose the stones have weights ``x`` and ``y`` with ``x <= y``.  The result of this smash is:
- If ``x == y``, both stones are totally destroyed;
- If ``x != y``, the stone of weight ``x`` is totally destroyed, and the stone of weight ``y`` has new weight ``y-x``.

At the end, there is at most ``1`` stone left.  Return the **smallest possible** weight of this stone (the weight is 0 if there are no stones left.)
### Example 1:
```c++
Input: [2,7,4,1,8,1]
Output: 1
Explanation: 
We can combine 2 and 4 to get 2 so the array converts to [2,7,1,8,1] then,
we can combine 7 and 8 to get 1 so the array converts to [2,1,1,1] then,
we can combine 2 and 1 to get 1 so the array converts to [1,1,1] then,
we can combine 1 and 1 to get 0 so the array converts to [1] then that's the optimal value.
```
### Note:
1. ``1 <= stones.length <= 30``
2. ``1 <= stones[i] <= 100``

# 解法
&emsp;题目中给了一些石头，要求把它们smash，两个石头重量相等时，碰撞后消失，不等时留下重量等于它们差值的石头。  
&emsp;这道题本质上其实是对所有石头重量前赋予正负号，需要求怎么赋予正负号最后的和绝对值最小。  这里把符号为正的石头当作一堆，符号为负的当作另一堆，接下来就是怎么分堆了，假设所有石头总重量为sum, 肯定是某堆重量越接近sum/2时和越小。接下来就来求这个能分出来的接近sum/2的堆。  
&emsp;那就是动态规划了，假设重量和为sum,需要求出所有能分出来的堆重量。动态数组dp[i]为true时表示重量i可以得到，为false表示不可得到。初始化全为false，dp[0]=true。依次把每个石头加到所有已经得到的堆里面即可得到所有能分出的堆，dp[j] = dp[j] | dp[j - stones[i]]。  
&emsp;最终从sum/2往0遍历，第一个true就是最接近sum/2的堆，sum - i - i就是所求。
```c++
int lastStoneWeightII(vector<int>& stones) 
{
	int n = stones.size(), sum = 0;
	for (int& stone : stones)
		sum += stone;
	vector<bool> dp(sum + 1, false);
	dp[0] = true;
	
	for (int i = 0; i < n; ++i)
	{
		for (int j = sum / 2; j >= stones[i]; --j)
			dp[j] = dp[j] | dp[j - stones[i]];
	}

	for (int i = sum / 2; i >= 0; --i)
	{
		if (dp[i])
			return sum - i - i;
	}
	return sum;
}
```
time complexity: O(n * sum)  
space complexity: O(sum)  
runtime: 100.00%  
memory usage: 100.00%  

# Link
[leetcode-1049](https://leetcode.com/problems/last-stone-weight-ii/)