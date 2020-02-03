---
layout: post
title: "Leetcode646-Maximum Length of Pair Chain"
subtitle: 'Leetcode-dynamic programming, greedy'
date:       2019-11-14 10:42:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - greedy
---

# 问题描述
You are given ``n`` pairs of numbers. In every pair, the first number is always smaller than the second number.

Now, we define a pair ``(c, d)`` can follow another pair ``(a, b)`` if and only if`` b < c``. Chain of pairs can be formed in this fashion.

Given a set of pairs, find the length longest chain which can be formed. You needn't use up all the given pairs. You can select pairs in any order.
### Example 1:
```c++
Input: [[1,2], [2,3], [3,4]]
Output: 2
Explanation: The longest chain is [1,2] -> [3,4]
```
### Note:
1. The number of given pairs will be in the range [1, 1000].

# 解法
## 解法 1
这题刚开始一看感觉无法用贪心，假设数组为[1, 10], [2, 3], [4, 5], [6, 7]，如果贪心就认为[1, 10]符合，返回了1,正确应该是3。那么只能动态规划了，动态规划就适合这种遍历所有情形的运算。因为要比较后面0位置和前面1位置，所以要先排序。动态数组dp长度为数组长度n，初始化全为1。dp[i]表示到第i+1个pair最大链的长度，动态转移方程就是对每个pair，pairs[i]，找到它之前pairs[j][1]小于pairs[i][0]的j位置长度加1，即dp[i] = max(dp[i], dp[j] + 1)。最终返回dp[n-1]。
```c++
int findLongestChain(vector<vector<int>>& pairs)
{
	int n = pairs.size();
	sort(pairs.begin(), pairs.end(), [](vector<int>& a, vector<int>& b) {if (a[0] == b[0]) return a[1] < b[1]; return a[0] < b[0];});
	vector<int> dp(n, 1);
	for (int i = 0; i < n; ++i)
	{
		for (int j = i - 1; j >= 0; --j)
		{
			if (pairs[j][1] < pairs[i][0]){
				dp[i] = max(dp[i], dp[j] + 1);
			}
		}
	}
	return dp.back();
}
```
timecomplexity: O(n^2)  
space complexity: O(n)  
runtime: 39.58%  
memory usage: 100.00%  

## 解法 2
这题刚开始感觉无法使用贪心，但看了大佬的解法还是太naive。因为题目限定对每个pair，pair[0] < pair[1]。那么这里排序是只按照1位置排序，这个相当关键，因为按照1位置排序就同时按照了0位置排序。再用一个栈存储上个pair，当前pair[0] > st.top()[1]就推入。回到解法1的例子，排序后为[2, 3], [4, 5], [6, 7]，[1, 10]。最终栈内为[2, 3], [4, 5], [6, 7]。
```c++
int findLongestChain(vector<vector<int>>& pairs)
{
	stack<vector<int>> st;
	sort(pairs.begin(), pairs.end(), [](vector<int>& a, vector<int>& b) {return a[1] < b[1];});
	for (auto& pair : pairs)
	{
		if (st.empty())
			st.push(pair);
		else {
			vector<int> t = st.top();
			if (pair[0] > t[1])
				st.push(pair);
		}
	}
	return st.size();
}
```
timecomplexity: O(n*log(n))  
space complexity: O(n)  
runtime: 73.04%  
memory usage: 66.67%  

## 解法 3
解法2空间优化。因为当前pair仅与上个pair的1位置元素有关，用end存储，同时不用栈了，直接用res表示大小。
```c++
int findLongestChain(vector<vector<int>>& pairs)
{
	int res = 0, end = INT_MIN;
	sort(pairs.begin(), pairs.end(), [](vector<int>& a, vector<int>& b) {return a[1] < b[1];});
	for (auto& pair : pairs)
	{
		if (pair[0] > end){
			++res;
			end = pair[1];
		}
	}
	return res;
}
```
timecomplexity: O(n*log(n))  
space complexity: O(1)  
runtime: 84.37%  
memory usage: 100.00%  

# Link
[leetcode-646](https://leetcode.com/problems/maximum-length-of-pair-chain/)  
[solution 2,3](https://www.cnblogs.com/grandyang/p/7381633.html)  