---
layout: post
title: "Leetcode446 - Arithmetic Slices II - Subsequence"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-20 18:11:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
A sequence of numbers is called arithmetic if it consists of at least three elements and if the difference between any two consecutive elements is the same.  
For example, these are arithmetic sequences:
```c++
1, 3, 5, 7, 9
7, 7, 7, 7
3, -1, -5, -9
```

The following sequence is not arithmetic.
```c++
1, 1, 2, 5, 7
```
A zero-indexed array A consisting of N numbers is given. A **subsequence** slice of that array is any sequence of integers (``P0, P1, ..., Pk``) such that ``0 ≤ P0 < P1 < ... < Pk < N``.

A **subsequence** slice (``P0, P1, ..., Pk``) of array A is called arithmetic if the sequence ``A[P0], A[P1], ..., A[Pk-1], A[Pk]`` is arithmetic. In particular, this means that ``k ≥ 2``.

The function should return the number of arithmetic subsequence slices in the array A.

The input contains N integers. Every integer is in the range of ``-2^31`` and ``2^31-1`` and ``0 ≤ N ≤ 1000``. The output is guaranteed to be less than ``2^31-1``.

### Example 1:
```c++
Input: [2, 4, 6, 8, 10]

Output: 7

Explanation:
All arithmetic subsequence slices are:
[2,4,6]
[4,6,8]
[6,8,10]
[2,4,6,8]
[4,6,8,10]
[2,4,6,8,10]
[2,6,10]
```

# 解法
这种求最值的题目不是动态规划就是贪心，这题要求全局解所以采用动态规划解法。  
首先需要解决动态数组如何定义的问题，如果定义为dp[i]表示到i位置subsequences的个数，那么状态转移方程无法求解，因为当前位置的subsequence和前面所有位置的subsequence有关。需要第二位存储前面的序列信息。首先想到的是定义第二维为diff的长度，但diff可以达到2 * INT_MAX，消耗内存过多，需要采用哈希表存储各个位置的diff信息。所以现在的动态数组为vector<int, unordered_map<int, int>>。  
首先想的是dp[i][diff]存储位置i差值为diff的序列长度，和[leetcode-413](https://hexinlin.top/2019/11/20/leetcode-413-Arithmetic-Slices/)一样，但这题有许多重复数字，或者diff==0，难以code。这里采用dp[i][diff]表示到i位置diff的个数。  
对每个i位置，遍历前面的位置j从0到i-1, diff = A[i] - A[j], diff[i] += 1(这里很重要，之后解释), 如果dp[j].count(diff)存在表示j之前出现过这个diff，res += dp[j][diff], dp[i][diff] += dp[j][diff]。这里和[leetcode-413](https://hexinlin.top/2019/11/20/leetcode-413-Arithmetic-Slices/)思想一样，只是不存长度改为diff个数，当前diff加进来，假设序列长度为interval, 新加subsequnce个数为interval-2，同时对应i位置diff有interval-1个，j位置diff有interval-2个，所以 res += dp[j][diff]和前面思想一模一样。  
再解释为啥dp[i][diff]要自增1，一是为了加diff[j][diff]，而第二点更重要的当有重复数字时dp[i][diff]自增在统计重复个数，举例子{2, 2, 3, 4},如果不是自增，就返回了1，正确为2.  
注意下面代码中先用long long存储差值，如果delta不在int表示范围内，则continue，为啥？因为题目限定每个数字在int范围内，如果delta不在int范围内，是没法组成大于等于3个的subsequences。

```c++
int numberOfArithmeticSlices(vector<int>& A)
{
	int res = 0, n = A.size();
	vector<unordered_map<int, int>> dp(n);
	for (int i = 1; i < n; ++i)
	{
		for (int j = 0; j < i; ++j)
		{
			long long delta = (long long)A[i] - A[j];
			if (delta > INT_MAX || delta < INT_MIN)
				continue;
			int diff = (int)delta;
			++dp[i][diff];
			if (dp[j].count(diff)){
				dp[i][diff] += dp[j][diff] ;
				res += dp[j][diff];
			}
		}
	}
	return res;
}
```
time complexity: O(n^2)  
space complexity: O(n)  
runtime: 85.61%  
memory usage: 80.00%  

# Link
[leetcode-446](https://leetcode.com/problems/arithmetic-slices-ii-subsequence/)   
[solution ](https://www.cnblogs.com/grandyang/p/6057934.html)  
[大神fun4LeetCode解释](https://leetcode.com/problems/arithmetic-slices-ii-subsequence/discuss/92822/Detailed-explanation-for-Java-O(n2)-solution)
