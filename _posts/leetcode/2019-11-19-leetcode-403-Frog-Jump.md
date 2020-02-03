---
layout: post
title: "Leetcode403 - Frog Jump"
subtitle: 'Leetcode-dynamic programming, recursion'
date:       2019-11-19 15:15:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - recursion
---

# 问题描述
A frog is crossing a river. The river is divided into x units and at each unit there may or may not exist a stone. The frog can jump on a stone, but it must not jump into the water.

Given a list of stones' positions (in units) in sorted ascending order, determine if the frog is able to cross the river by landing on the last stone. Initially, the frog is on the first stone and assume the first jump must be 1 unit.

If the frog's last jump was k units, then its next jump must be either ``k - 1``, ``k``, or ``k + 1`` units. Note that the frog can only jump in the forward direction.

### Note:
- The number of stones is ``≥ 2`` and is ``< 1,100``.
- Each stone's position will be a non-negative integer ``< 2^31``.
- The ``first stone``'s position is always ``0``.

### Example 1:
```c++
[0,1,3,5,6,8,12,17]

There are a total of 8 stones.
The first stone at the 0th unit, second stone at the 1st unit,
third stone at the 3rd unit, and so on...
The last stone at the 17th unit.

Return true. The frog can jump to the last stone by jumping 
1 unit to the 2nd stone, then 2 units to the 3rd stone, then 
2 units to the 4th stone, then 3 units to the 6th stone, 
4 units to the 7th stone, and 5 units to the 8th stone.
```
### Example 2:
```c++
[0,1,2,3,4,8,9,11]

Return false. There is no way to jump to the last stone as 
the gap between the 5th and 6th stone is too large.
```

# 解法
## 解法 1
青蛙过河，假设上次跳了k距离，这次只能跳k-1, k和k+1距离，问能否过河。  
首先想的是动态规划解法，对每个石头，遍历青蛙它前面的石头时跳的距离，观察这个距离能否达到（-1，+1，+0）当前石头。
如果能够达到，把当前跳的距离存入当前位置动态数组，所以首先想的是用二维动态数组存储之前跳的距离。  
dp[i]为vector<int>表示达到第i块石头是跳的距离。解法如下：
```c++
    bool canCross(vector<int>& stones) {
        int n = stones.size();
        vector<vector<unsigned int>> dp(n);
        dp[0] = {0};
        for (int i = 1; i < n; ++i)
        {
            for (int j = i - 1; j >= 0; --j)
            {
                if (!dp[j].empty() && dp[j].back() + 1 + stones[j] >= stones[i]){
                    for (int k = 0; k < dp[j].size(); ++k)
                    {
                        if (dp[j][k] > 1 && dp[j][k] - 1 + stones[j] == stones[i]){
                            dp[i].push_back(dp[j][k] - 1);
                            if (i == n - 1)
                                return true;
                        }

                        if (dp[j][k] >= 1 && dp[j][k] + stones[j] == stones[i]){
                            dp[i].push_back(dp[j][k]);
                            if (i == n - 1)
                                return true;
                        }
                        if (dp[j][k] >= 0 && dp[j][k] + 1 + stones[j] == stones[i]){
                            dp[i].push_back(dp[j][k] + 1);
                            if (i == n - 1)
                                return true;
                        }					
                    }
                }
            }
            sort(dp[i].begin(), dp[i].end());
        }
        return false;     
    }
```
但这个解法超时了，因为对每个位置存距离是vector<int>，有许多重复值，而集合可以去除重复值，所以考虑用vector<int, set<int>>表示动态数组，dp[i]为有序集合，根据末尾还能判断当前石头能否到达所选位置，很方便。
```c++
bool canCross(vector<int>& stones)
{
	int n = stones.size();
	vector<set<unsigned int>> dp(n);
	dp[0] = {0};
	for (int i = 1; i < n; ++i)
	{
		for (int j = i - 1; j >= 0; --j)
		{
			if (!dp[j].empty() && *(dp[j].rbegin()) + 1 + stones[j] >= stones[i]){
				for (set<unsigned int>::iterator it = dp[j].begin(); it != dp[j].end(); ++it)
				{
					if (*it - 1 + stones[j] == stones[i]){
						dp[i].insert(*it - 1);
					}

					if (*it + stones[j] == stones[i]){
						dp[i].insert(*it);
					}
					if (*it + 1 + stones[j] == stones[i]){
						dp[i].insert(*it + 1);
					}										
				}
			}
		}
	}
	if (dp.back().empty())
		return false;
	return true;
}
```
runtime: 15.69%   
memory usage: 30.00% 

## 解法 2
解法1动态规划存在许多次重复插入，这里用dp[i]表示青蛙达到位置i石头时的跳跃能力，超越跳跃能力就需往后遍历。
用哈希表unordered_map<int, unordered_set<int>>存储到达位置i时的所有跳跃距离，还是用集合去重。
```c++
bool canCross3(vector<int>& stones)
{
	unordered_map<int, unordered_set<int>> m;
	m[0] = {0};
	int k = 0, n = stones.size();
	vector<int> dp(n, 0);
	for (int i = 1; i < n; ++i)
	{
		while (dp[k] + 1 < stones[i] - stones[k])
			++k;
		for (int j = k; j < i; ++j)
		{
			int dis = stones[i] - stones[j];
			if (m[j].count(dis - 1) || m[j].count(dis) || m[j].count(dis + 1)){
				m[i].insert(dis);
				dp[i] = max(dp[i], dis);
			}
		}
	}
	return dp.back() > 0;
}
```
runtime:  65.48%  
memory usage: 15.00%  


## 解法 3
动态规划带记忆数组的递归写法。用哈希表存储每个位置和跳跃距离的状态，考虑用{pos, jump}当作键，但vector<int>是不可哈希的，所以要想
办法把pos,jump合二为一。这里``key = pos | jump << 11``。
```c++
bool helper(vector<int>& stones, int pos, int jump, unordered_map<int, bool>& m)
{
	if (pos >= stones.size() - 1)
		return true;
	int key = pos | (jump << 11);
	if (m.count(key))
		return m[key];
	for (int i = pos + 1; i < stones.size(); ++i)
	{
		int distance = stones[i] - stones[pos];
		if (jump + 1 < distance)
			return m[key] = false;
		if (jump - 1 > distance)
			continue;
		if (helper(stones, i, distance, m))
			return m[key] = true;
	}
	return m[key] = false;
}

bool canCross2(vector<int>& stones)
{
	unordered_map<int, bool> m;
	return helper(stones, 0, 0, m);
}
```
runtime: 83.14%  
memory usage: 50.00%  

# Link
[leetcode-403](https://leetcode.com/problems/frog-jump/)  
[solution 2, 3](https://www.cnblogs.com/grandyang/p/5888439.html)  