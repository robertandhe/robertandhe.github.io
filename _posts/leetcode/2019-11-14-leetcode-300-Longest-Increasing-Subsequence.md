---
layout: post
title: "Leetcode300-Longest Increasing Subsequence"
subtitle: 'Leetcode-dynamic programming, binary search'
date:       2019-11-14 11:37:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - binary search
---

# 问题描述
Given an unsorted array of integers, find the length of longest increasing subsequence.
### Example:
```c++
Input: [10,9,2,5,3,7,101,18]
Output: 4 
Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4. 
```
### Note:
- There may be more than one LIS combination, it is only necessary for you to return the length.
- Your algorithm should run in O(n2) complexity.
### Follow up:
Could you improve it to O(n log n) time complexity?

# 解法
## 解法 1
这题一看就和[leetcode-646](https://hexinlin.top/2019/11/14/leetcode-646-Maximum-Length-of-Pair-Chain/)思路一样。
```c++
int lengthOfLIS(vector<int>& nums)
{
	if (nums.empty())
			return 0;
	int res = 1, n = nums.size();
	vector<int> dp(n, 1);
	for (int i = 0; i < n; ++i)
	{
		for (int j = i - 1; j >= 0; --j)
		{
			if (nums[j] < nums[i])
				dp[i] = max(dp[i], dp[j] + 1);		
		}
		res = max(res, dp[i]);
	}
	return res;
}
```
time complexity: O(n^2)  
space complexity: O(n)  
runtime: 15.74%  
memory usage:  75.00%  

## 解法 2
题目follow up提示有O(n*logn)解法，那么应该是二分法。不过不会，网上找找。这个解法维持一个最长长度的数组ends，里面的元素可能不是原始的LIS。ends初始化为nums[0],后面出现比ends[0]小的元素，就用它替换ends[0],如果比ends.back()大，就推入数组，长度增大，如果比开头小，末尾大，就用它替换数组中大于等于它的元素，这些操作背后的思想都是尽量缩小数组中的元素，以使后面长度尽量增长。这里查找大于等于它的位置就是二分法。
```c++
int lengthOfLIS(vector<int>& nums)
{
	if (nums.empty())
		return 0;
	vector<int> ends{nums[0]};
	for (int& a : nums)
	{
		if (a < ends[0])
			ends[0] = a;
		else if (a > ends.back())
			ends.push_back(a);
		else {
			int left = 0, right = ends.size();
			while (left < right){
				int mid = left + (right - left) / 2;
				if (ends[mid] < a)
					left = mid + 1;
				else 
					right = mid;
			}
			ends[right] = a;
		}
	}
	return ends.size();
}
```
time complexity: O(n*log(n))  
space complexity: O(n)  
runtime: 90.31%  
memory usage: 95.31%  

## 解法 3
和解法2一样，代码更加浑然一体。
```c++
int lengthOfLIS3(vector<int>& nums)
{
	if (nums.empty())
		return 0;
	vector<int> ends{nums[0]};
	for (int& a : nums)
	{
		int left = 0, right = ends.size();
		while (left < right){
			int mid = left + (right - left) / 2;
			if (ends[mid] < a)
				left = mid + 1;
			else 
				right = mid;
		}
		if (right >= ends.size())
			ends.push_back(a);
		else 
			ends[right] = a;
	}
	return ends.size();
}
```
time complexity: O(n*log(n))  
space complexity: O(n) 
runtime: 90.31%  
memory usage: 48.44%  

## 解法 4
和解法2，3一样，只是用STL lower_bound来查找。
```c++
int lengthOfLIS(vector<int>& nums)
{
	if (nums.empty())
		return 0;
	vector<int> ends{nums[0]};
	for (int& a : nums)
	{
		auto it = lower_bound(ends.begin(), ends.end(), a);
		if (it == ends.end())
			ends.push_back(a);
		else 
			*it = a;
	}
	return ends.size();
}
```
time complexity: O(n*log(n))  
space complexity: O(n)  
runtime: 90.31%  
memory usage: 95.31%  

# Link
[leetcode-300](https://leetcode.com/problems/longest-increasing-subsequence/)  
[solution 2-4](https://www.cnblogs.com/grandyang/p/4938187.html)  