---
layout: post
title: "Leetcode416-Partition Equal Subset Sum"
subtitle: 'Leetcode-dynamic programming, bit manipulation'
date:       2019-11-02 16:54:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - bit manipulation
---

# 问题描述
Given a non-empty array containing only positive integers, find if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal.
### Note:
1. Each of the array element will not exceed 100.
2. The array size will not exceed 200.

### Example 1:
```c++
Input: [1, 5, 11, 5]

Output: true

Explanation: The array can be partitioned as [1, 5, 5] and [11].
```
### Example 2:
```c++
Input: [1, 2, 3, 5]

Output: false

Explanation: The array cannot be partitioned into equal sum subsets.
```

# 解法
## 解法 1
这道题要求把数组分成两部分，每部分的和相等。为了判断某个和是否出现，应选用dp来做。数组和应该为偶数，target为数组和的一半。
建立target+1长度的数组dp,dp[0]设为true, dp[i]为1表示和i能够出现，那么关键就是建立状态转移方程。在遍历过程中，对每个遍历到的数num，dp[j]是否为1，要看现在的dp[j]和dp[j-num]，即dp[j] = dp[j] || dp[j - num]。
```c++
bool canPartition(vector<int>& nums)
{
	int sum = 0;
	for (int& num : nums)
		sum += num;
	if (sum % 2)
		return false;
	int target = sum / 2;
	vector<bool> dp(target + 1);
	dp[0] = true;
	for (int& num : nums)
	{
		for (int i = target; i >= num; --i)
			dp[i] = dp[i] || dp[i - num];
	}
	return dp[target];
}
```
time complexity: O(n)  
space complexity: O(m)  
runtime: 25.73%  
memory usage: 94.12%   

## 解法 2
利用bitset，长度为10001，因为nums最长为200，nums中每个数不超过100，除2加个0，就为10001。bitset初始化为1，遍历到num时，bs |= bs << num，每个和对应的位置都为1，最后判断bs[target]。
```c++
bool canPartition(vector<int>& nums)
{
	int sum = 0;
	for (int& num : nums)
		sum += num;
	bitset<10001> bs(1);
	for (int& num : nums)
	{
		bs |= bs << num;
	}
	return (sum % 2 == 0) && bs[sum / 2];
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 87.03%  
memory usage: 94.12%  

# link
[leetcode-416](https://leetcode.com/problems/partition-equal-subset-sum/) 
[solution](https://www.cnblogs.com/grandyang/p/5951422.html)