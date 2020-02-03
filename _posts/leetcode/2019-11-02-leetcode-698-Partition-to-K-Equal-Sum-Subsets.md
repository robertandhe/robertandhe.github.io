---
layout: post
title: "Leetcode698-Partition to K Equal Sum Subsets"
subtitle: 'Leetcode-recursion'
date:       2019-11-02 18:32:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - recursion
---

# 问题描述
Given an array of integers nums and a positive integer ``k``, find whether it's possible to divide this array into ``k ``non-empty subsets whose sums are all equal.

### Example 1:
```c++
Input: nums = [4, 3, 2, 3, 5, 2, 1], k = 4
Output: True
Explanation: It's possible to divide it into 4 subsets (5), (1, 4), (2,3), (2,3) with equal sums.
```
### Note:
- ``1 <= k <= len(nums) <= 16.``
- ``0 < nums[i] < 10000.``

# 解法
## 解法 1
典型递归问题。先求数组和，然后看是否能整除k，不能则返回``false``。``target``为``sum``除``k``，建立``visited``数组存储是否访问过，``start``表示开始位置，``curSum``表示当前和。递归时当前``k==1``，表示前面``k-1``个子集均已找到，剩下的肯定也满足和为``target``，返回``true``。当``curSum > target``时，没有继续的必要，返回``false``。当``curSum == target``,表示已经找到一个子集和为``target``，则k减1，开始下次递归，注意这里``start``要从0开始，因为上次没访问的位置，下次递归时可能需要。
```c++
bool helper(vector<int>& nums, int& target, int k, int start, int curSum, vector<bool>& visited)
{
	if (k == 1)
		return true;
	if (curSum > target)
		return false;	
	if (curSum == target)
		return helper(nums, target, k - 1, 0, 0, visited);
	for (int i = start; i < nums.size(); ++i)
	{
		if (visited[i])
			continue;
		visited[i] = true;
		if (helper(nums, target, k, i + 1, curSum + nums[i], visited))
			return true;
		visited[i] = false;
	}
	return false;
}

bool canPartitionKSubsets(vector<int>& nums, int k)
{
	int sum = accumulate(nums.begin(), nums.end(), 0), n = nums.size();
	if (sum % k)
		return false;
	vector<bool> visited(n, false);
	int start = 0, curSum = 0, target = sum / k;
	return helper(nums, target, k, start, curSum, visited);
}
```
runtime: 64.02%  
memory usage: 75.00%   

# 解法 2
也是递归。用长度为``k``的数组``v``存储各个自己的和。``idx``表示当前访问的位置，如果当前``nums[idx]``加到``v[i] <= target``则可以加上去试试，否则访问``v[i+1]``，注意这里应该先给``nums``排序，能加快运行速度。``idx == -1``时，表示已经访问完``nums``所有元素，此时判断``v``是否都等于``target``。
```c++
bool helper(vector<int>& nums, vector<int>& v, int target, int idx)
{
	if (idx == -1){
		for (int i = 0; i < v.size(); ++i)
		{
			if (v[i] != target)
				return false;
		}
		return true;
	}
	int num = nums[idx];
	for (int i = 0; i < v.size(); ++i)
	{
		if (v[i] + num > target)
			continue;
		v[i] += num;
		if (helper(nums, v, target, idx - 1))
			return true;
		v[i] -= num;
	}
	return false;
}

bool canPartitionKSubsets(vector<int>& nums, int k)
{
	int sum = accumulate(nums.begin(), nums.end(), 0), n = nums.size();
	if (sum % k)
		return false;
	vector<int> v(k, 0);
	sort(nums.begin(), nums.end());
	int idx = n - 1;
	return helper(nums, v, sum / k, idx);
}
```
runtime: 27.61%  
memory usage: 95.00%   

# link
[leetcode-698](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/)