---
layout: post
title: "Leetcode740-Delete and Earn"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-23 11:23:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given an array ``nums`` of integers, you can perform operations on the array.

In each operation, you pick any ``nums[i]`` and delete it to earn ``nums[i]`` points. After, you must delete **every** element equal to ``nums[i] - 1`` or ``nums[i] + 1``.

You start with 0 points. Return the maximum number of points you can earn by applying such operations.

### Example 1
```c++
Input: nums = [3, 4, 2]
Output: 6
Explanation: 
Delete 4 to earn 4 points, consequently 3 is also deleted.
Then, delete 2 to earn 2 points. 6 total points are earned.
```
### Example 2
```c++
Input: nums = [2, 2, 3, 3, 3, 4]
Output: 9
Explanation: 
Delete 3 to earn 3 points, deleting both 2's and the 4.
Then, delete 3 again to earn 3 points, and 3 again to earn 3 points.
9 total points are earned.
```
### Note:
- The length of ``nums`` is at most`` 20000``.
- Each element ``nums[i]`` is an integer in the range [``1, 10000``].

# 解法
## 解法 1
这题和之前那道[leetcode 198. House Robber](https://leetcode.com/problems/house-robber/)很相似，那里不能同时偷相邻的房屋，这里不是保留相邻的数。这里可能存在重复数字，那么就需要统计每个数字出现的次数，同时要根据相邻数字来进行状态转移，那么每个数字一个桶正好能解决这个问题。对每个数字要分保留还是丢弃，保留的话就是前一个不保留加这个位置的数字总和Earn[i] = notEarn[i - 1] + i * dp[i]，丢弃的话就是就是前一个保留和丢弃中的较大值。
```c++
int deleteAndEarn(vector<int>& nums)
{
	if (nums.empty())	
		return 0;
	int mx = 0;
	for (int& num : nums)
		mx = max(mx, num);
	vector<int> dp(mx + 1, 0), Earn(mx + 1, 0), notEarn(mx + 1, 0);
	for (int& num : nums)
		++dp[num];
	for (int i = 1; i <= mx; ++i)
	{
		Earn[i] = notEarn[i - 1] + i * dp[i];
		notEarn[i] = max(Earn[i - 1], notEarn[i - 1]);
	}
	return max(Earn.back(), notEarn.back());
}
```
timecomplexity: O(n)  
space complexity: O(mx)  
runtime: 99.12%  
memory usage: 75.00%  


## 解法 2
和解法一一样，只是空间优化了。
```c++
int deleteAndEarn(vector<int>& nums)
{
	if (nums.empty())	
		return 0;
	int mx = 0;
	for (int& num : nums)
		mx = max(mx, num);
	vector<int> dp(mx + 1, 0);
	for (int& num : nums)
		++dp[num];
	int take = 0, skip = 0;
	for (int i = 1; i <= mx; ++i)
	{
		int skipi = max(skip, take);
		int takei = skip + i * dp[i];
		take = takei;
		skip = skipi;
	}
	return max(take, skip);
}
```
timecomplexity: O(n)  
space complexity: O(mx)  
runtime: 99.12%  
memory usage: 100.00%  


## 解法 3
直接在nums数组上更新
```c++
int deleteAndEarn(vector<int>& nums)
{
	vector<int> sums(10001, 0);
	for (int& num : nums)
		sums[num] += num;
	for (int i = 2; i < 10001; ++i)
		sums[i] = max(sums[i - 1], sums[i - 2] + sums[i]);
	return sums.back();
}
```
timecomplexity: O(n)  
space complexity: O(1)  
runtime: 34.93%  
memory usage: 25.00%  

# Link
[leetcode740](https://leetcode.com/problems/delete-and-earn/)  
[solution 2&3](https://www.cnblogs.com/grandyang/p/8176933.html)