---
layout: post
title: "Leetcode763-Partition Labels"
subtitle: 'Leetcode763-greedy'
date:       2019-10-26 09:57:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
---

# 问题描述
A string ``S`` of lowercase letters is given. We want to partition this string into as many parts as possible so that each letter appears in at most one part, and return a list of integers representing the size of these parts.
### Example 1:
```c++
Input: S = "ababcbacadefegdehijhklij"
Output: [9,7,8]
Explanation:
The partition is "ababcbaca", "defegde", "hijhklij".
This is a partition so that each letter appears in at most one part.
A partition like "ababcbacadefegde", "hijhklij" is incorrect, because it splits S into less parts.
```
### Note:
1. ``S`` will have length in range ``[1, 500]``.
2. ``S`` will consist of lowercase letters ``('a' to 'z')`` only.

# 解法
根据题目的hint即可解题。首先一个partition里面的字符一定不再别的partition里面出现，现在问题就是怎么划分每个parition。首先应该找到第一个
字符的最后出现位置，这样第一个字符就包含在partition里面，但中间还有别的字符，所以要找到中间所有字符的最后出现位置。直到partition内所有字符
的最后位置均在partition范围内。用哈希表存储最后出现位置。
```c++
vector<int> partitionLabels(string S)
{
	vector<int> res;
	int first = 0, last = 0;
	unordered_map<char, int> m;
	for (int i = 0; i < S.size(); ++i)
		m[S[i]] = i;
	while (first < S.size()){
		last = m[S[first]];
		for (int j = first + 1; j <= last; ++j)
		{
			if (m[S[j]] > last){
				last = m[S[j]];
			}
		}
		res.push_back(last - first + 1);
		first = last + 1;
	}
	return res;
}
```
时间复杂度：O(n)  
空间复杂度：O(n)  
runtime: 92.31%  
memory usage: 70.97%  

# link
[leetcode-763](https://leetcode.com/problems/partition-labels/)