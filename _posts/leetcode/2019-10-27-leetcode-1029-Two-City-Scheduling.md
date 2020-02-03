---
layout: post
title: "Leetcode1029-Two City Scheduling"
subtitle: 'Leetcode1029-greedy, sort'
date:       2019-10-27 10:37:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
  - sort
---

# 问题描述
There are ``2N`` people a company is planning to interview. The cost of flying the ``i``-th person to city ``A`` is ``costs[i][0]``, and the cost of flying the i-th person to city ``B`` is ``costs[i][1]``.

Return the minimum cost to fly every person to a city such that exactly ``N`` people arrive in each city.
### Example 1:
```c++
Input: [[10,20],[30,200],[400,50],[30,20]]
Output: 110
Explanation: 
The first person goes to city A for a cost of 10.
The second person goes to city A for a cost of 30.
The third person goes to city B for a cost of 50.
The fourth person goes to city B for a cost of 20.

The total minimum cost is 10 + 30 + 50 + 20 = 110 to have half the people interviewing in each city.
```
### Note:
1. ``1 <= costs.length <= 100``
2. It is guaranteed that ``costs.length`` is even.
3. ``1 <= costs[i][0], costs[i][1] <= 1000``

# 解法
这道题虽然是easy难度，但还是挺有意思的。咋一看无法下手，感觉是看0，1那个小就放到那个A或者B，但同时还要是两个均等分布。其实这道题应该看0，1两个位置的差值，按照差值由小到大排序，前N个放到A，后N个放到B。因为我们关心的是开支，按照差值排使放到A效益最大化。
```c++
int twoCitySchedCost(vector<vector<int>>& costs)
{
	int res = 0, n = costs.size();
	auto cmp = [](vector<int>& a, vector<int>& b) {return a[0] - a[1] < b[0] - b[1];};
	sort(costs.begin(), costs.end(), cmp);
	for (int i = 0; i < n / 2; ++i){
		res += costs[i][0];
		res += costs[i + n / 2][1];
	}
	return res;	
}
```
时间复杂度：O(nlog(n))  
空间复杂度：O(1)  
runtime: 100.00%  
memory usage: 94.74%  

# link
[leetcode-1029](https://leetcode.com/problems/two-city-scheduling/)