---
layout: post
title: "Leetcode139-Word Break"
subtitle: 'Leetcode-dynamic programming, recursion, BFS'
date:       2019-11-07 10:24:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - recursion
  - BFS
---

# 问题描述
Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, determine if s can be segmented into a space-separated sequence of one or more dictionary words.
### Note:
- The same word in the dictionary may be reused multiple times in the segmentation.
- You may assume the dictionary does not contain duplicate words.

### Example 1:
```c++
Input: s = "leetcode", wordDict = ["leet", "code"]
Output: true
Explanation: Return true because "leetcode" can be segmented as "leet code".
```
### Example 2:
```c++
Input: s = "applepenapple", wordDict = ["apple", "pen"]
Output: true
Explanation: Return true because "applepenapple" can be segmented as "apple pen apple".
             Note that you are allowed to reuse a dictionary word.
```
### Example 3:
```c++
Input: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
Output: false
```

# 解法
## 解法 1
这道题属于经典的动态规划问题，同时动态规划也肯定能用带记忆数组的递归解法求解，如求解斐波那契数列问题。
1. 动态规划解法
```c++
int fib(int N)
{
	if (N == 0)
		return 0;
	if (N == 1)
		return 1;
	vector<int> dp(N + 1, 0);
	dp[0] = 0;
	dp[1] = 1;
	for (int i = 2; i <= N; ++i)
		dp[i] = dp[i - 1] + dp[i - 2];
	return dp[N];
}
```

2. 记忆数组递归解法
```c++
int helper(int N, unordered_map<int, int>& m)
{
	if (!m.count(N)){
		int t = helper(N - 1, m) + helper(N - 2, m);
		m[N] = t;
	}
	return m[N];
}
int fib(int N)
{
	unordered_map<int, int> m;
	if (N == 0)
		return 0;
	if (N == 1)
		return 1;
	m[0] = 0;
	m[1] = 1;
	return helper(N - 1, m) + helper(N - 2, m);
}
```

解法1为记忆数组递归解法，首先找一个可以在字典中找到的切分位置，然后对后面的字符串应用递归，所以仅当当前位置之前可找到才对后面应用递归。为了减少重复计算，用一个memo数组memo[i]表示i到n的字符串可分。为了在字典中加快查找，用哈希集unordered_set<string>存储所有字典，为常数平均查找时间。每次递归用start记录开始位置。
```c++
bool check(string s, int start, unordered_set<string>& st, vector<int>& memo)
{
	if (start == s.size())
		return true;
	if (memo[start] != -1)
		return memo[start];
	for (int i = start + 1; i <= s.size(); ++i)
	{
		if (st.count(s.substr(start, i - start)) && check(s, i, st, memo)){
			return memo[start] = 1;
		}
	}
	return memo[start] = 0;
}

bool wordBreak(string s, vector<string>& wordDict)
{
	unordered_set<string> st(wordDict.begin(), wordDict.end());
	vector<int> memo(s.size(), -1);
	int start = 0;
	return check(s, start, st, memo);
}
```

runtime: 19.63%  
memory usage: 26.41%  

## 解法 2
为动态规划解法。动态规划两大问题，动态规划数组定义与状态转移方程，这里数组dp肯定是定义为dp[i]表示前i个可以分离，而状态转移方程为
dp[i]为true当且在之前能找到位置j满足dp[j]==true且s.substr(j, i -j)在字典中。注意dp数组初始化时dp[0]==true。
```c++
bool wordBreak(string s, vector<string>& wordDict)
{
	unordered_set<string> word(wordDict.begin(), wordDict.end());
	vector<bool> dp(s.size() + 1, false);
	dp[0] = true;
	for (int i = 0; i <= s.size(); ++i)
	{
		for (int j = 0; j < i; ++j)
		{
			if (dp[j] && word.count(s.substr(j, i - j))){
				dp[i] = true;
			}
		}
	}
	return dp.back();
}
```

time complexity: O(n^2)  
space complexity: O(n)  
runtime: 76.45%  
memory usage: 50.94%  

## 解法 3
为广度优先搜索解法。用队列存储可以分离的位置，当队列q不为空时，从中取元素start开始循环，直至字符串尾部。如果在中间s.substr(start, i - start)可以在字典中找到，那么表示i之前也是可分离的，i推入队列。
```c++
bool wordBreak(string s, vector<string>& wordDict)
{
	unordered_set<string> word(wordDict.begin(), wordDict.end());
	vector<int> visited(s.size(), 0);
	queue<int> q{{0}};
	while (!q.empty()){
		int start = q.front();
		q.pop();
		if (!visited[start]){
			for (int i = start + 1; i <= s.size(); ++i)
			{
				if (word.count(s.substr(start, i - start))){
					if (i == s.size())
						return true;
					q.push(i);
				}
			}
			visited[start] = 1;
		}
	}
	return false;
}
```

runtime：46.62%  
memory usage: 49.06%  

# link
[leetcode-139](https://leetcode.com/problems/word-break/)  
[unordered_set](https://zh.cppreference.com/w/cpp/container/unordered_set)  