---
layout: post
title: "Leetcode377 - Combination Sum IV"
subtitle: 'Leetcode-dynamic programming, recursion'
date:       2019-11-18 20:46:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - recursion
---

# 问题描述
Given an integer array with all positive numbers and no duplicates, find the number of possible combinations that add up to a positive integer target.
### Example:
```c++
nums = [1, 2, 3]
target = 4

The possible combination ways are:
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)

Note that different sequences are counted as different combinations.

Therefore the output is 7.
```
### Follow up:
What if negative numbers are allowed in the given array?
How does it change the problem?
What limitation we need to add to the question to allow negative numbers?

# 解法
## 解法 1
纯递归，超时。
```c++
int combinationSum4(vector<int>& nums, int target)
{
	if (target < 0)
		return 0;
	if (target == 0)
		return 1;
	int res = 0;
	for (int& num : nums)
	{
		res += combinationSum42(nums, target - num);
	}
	return res;
}
```
## 解法 2
带记忆数组递归，内存超出，nums={4, 2, 1}, target = 32, 迭代过深。
```c++
int helper(vector<int>& nums, int target, vector<int> mem)
{
	if (mem[target] != -1)
		return mem[target];
	if (target == 0)
		return 1;
	int res = 0;
	for (int i = 0; i < nums.size(); ++i)
	{
		if (nums[i] <= target)
			res += helper(nums, target-nums[i], mem);
	}
	mem[target] = res;
	return res;
}

int combinationSum4(vector<int>& nums, int target)
{
	vector<int> mem(target + 1, -1);
	return helper(nums, target, mem);
}
```

## 解法 3
只有上动态规划了。这题动态数组dp[i]表示和为i的子数组个数，那么状态转移方程为if i >= num, dp[i] += dp[i - num],因为i可由别的数加和得到。
如题目中nums = {1, 2, 3}, target = 4, dp[4] = dp[3](4-1) + dp[2](4-2) + d[1](4-3)。
```c++
    int combinationSum4(vector<int>& nums, int target)
    {
        vector<unsigned int> dp(target + 1);
        dp[0] = 1;
        for (int i = 1; i <= target; ++i)
        {
            for (int& num : nums)
            {
                if (i >= num)
                    dp[i] += dp[i - num];
            }
        }
        return dp[target];
    }
```
time complexity: O(target * n)  
space complexity: O(target)  
runtime: 100.00%  
memory usage:  100.00%  

# Link
[leetcode-377](https://leetcode.com/problems/combination-sum-iv/)  
[solution](https://leetcode.com/problems/combination-sum-iv/discuss/85036/1ms-Java-DP-Solution-with-Detailed-Explanation)  