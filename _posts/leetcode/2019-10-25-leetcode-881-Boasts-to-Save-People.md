---
layout: post
title: "Leetcode881-Boats to Save People"
subtitle: 'Leetcode881-greedy'
date:       2019-10-25 21:53:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
---

# 问题描述
The ``i``-th person has weight ``people[i]``, and each boat can carry a maximum weight of limit.

Each boat carries at most 2 people at the same time, provided the sum of the weight of those people is at most ``limit``.

Return the minimum number of boats to carry every given person.  (It is guaranteed each person can be carried by a boat.)  
### example 1:
```c++
Input: people = [1,2], limit = 3
Output: 1
Explanation: 1 boat (1, 2)
```
### example 2:
```c++
Input: people = [3,2,2,1], limit = 3
Output: 3
Explanation: 3 boats (1, 2), (2) and (3)
```
### example 3:
```c++
Input: people = [3,5,3,4], limit = 5
Output: 4
Explanation: 4 boats (3), (3), (4), (5)
```
### Note:
- ``1 <= people.length <= 50000``
- ``1 <= people[i] <= limit <= 30000``

# 解法
每艘船最多装两人，所以尽量最胖搭配最瘦的，至于能不能搭就要看两人体重之和在不在limit范围内,为贪心算法。所以要先排序，然后用双指针指向最胖和最瘦的人，最胖是一定要上船的，根据体重和的大小确定最瘦的上不上。
```c++
int numRescueBoats(vector<int>& people, int limit)
{
	int res = 0, n = people.size();
	sort(people.begin(), people.end());
	int left = 0, right = n - 1;
	while (left <= right){
		if (people[left] + people[right] <= limit)
			++left;
		--right;
		++res;
	}
	return res;
}
```
时间复杂度：O(nlogn)  
时间复杂度：O(1)  
runtime: 13.42%  
memory usage: 100.00%   

# link
[leetcode 881](https://leetcode.com/problems/boats-to-save-people/)