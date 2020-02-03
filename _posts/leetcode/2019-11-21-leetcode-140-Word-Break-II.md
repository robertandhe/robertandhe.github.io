---
layout: post
title: "Leetcode140-Word Break II"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-21 22:16:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, add spaces in s to construct a sentence where each word is a valid dictionary word. Return all such possible sentences.
### Note:
- The same word in the dictionary may be reused multiple times in the segmentation.
- You may assume the dictionary does not contain duplicate words.

### Example 1:
```c++
Input:
s = "catsanddog"
wordDict = ["cat", "cats", "and", "sand", "dog"]
Output:
[
  "cats and dog",
  "cat sand dog"
]
```
### Example 2:
```c++
Input:
s = "pineapplepenapple"
wordDict = ["apple", "pen", "applepen", "pine", "pineapple"]
Output:
[
  "pine apple pen apple",
  "pineapple pen apple",
  "pine applepen apple"
]
Explanation: Note that you are allowed to reuse a dictionary word.
```
### Example 3:
```c++
Input:
Input:
s = "catsandog"
wordDict = ["cats", "dog", "sand", "and", "cat"]
Output:
[]
```

# 解法
## 解法 1
这题是之前那道[leetcode139-Word Break](https://leetcode.com/problems/word-break/)的升级版，那里只要求求出能否break，这里要给出所有break的组合。  
首先想了广度优先搜索解法，用一个队列存储所有能够找到的字符串，同时为了往后找，应该记录当前字符串末尾对应s中的位置，这里用pair存储。为了加快在wordDict中查找的速度，用unordered_set存储加快查找。对每个pair，从pair.second往后找，如果到某个位置的子串能够在dict中找到，就加上去...
```c++
vector<string> wordBreak(string s, vector<string>& wordDict)
{
	int maxLen = 0;
	for (string& word : wordDict)
		maxLen = max(maxLen, (int)word.size());
	vector<string> res;
	unordered_set<string> st(wordDict.begin(), wordDict.end());
	queue<pair<string, int>> q;
	q.push({"", 0});
	int n = s.size();
	while (!q.empty()){
		auto p = q.front();
		q.pop();
		for (int i = p.second; i < n && i <= p.second + maxLen; ++i)
		{
			string t = s.substr(p.second, i - p.second + 1);
			if (st.count(t)){
				string m = "";
				if (p.first.size() == 0)
					m += t;
				else 
					m = p.first + " " + t;
				if (i == n - 1)
					res.push_back(m);
				else 
					q.push({m, i + 1});
			}
		}
	}
	return res;
}
```
这个解法是可行的，可是只能够通过3/4的测试，TLE了。

## 解法 2
这道题还是适合递归解法，递归能够很好遍历所有情况，这里以dict中的word为遍历点，而不是s中，和解法1有很大不同。在当前s中，对每个word如果能够在s开头word.size()找到，则可以递归到下层，s减去word的长度，对下层递归返回的vector<string>每个加上word还有空格就是这层递归要返回的。为了减少重复计算，应该用哈希表存储s对应string向量，递归开始时如果能在哈希表找到直接返回即可。还有边界情况s为空串了，这是应该返回空串，而且上层不加空格。
```c++
vector<string> helper(string s, vector<string>& wordDict, unordered_map<string, vector<string>>& m)
{
	if (m.count(s))
		return m[s];
	if (s.empty())
		return {""};
	vector<string> res;
	for (string word : wordDict)
	{
		if (s.substr(0, word.size()) != word)
			continue;
		vector<string> tmp = helper(s.substr(word.size()), wordDict, m);
		for (string str : tmp)
			res.push_back(word + (str.empty() ? "" : " ") + str);
	}
	return m[s] = res;
}

vector<string> wordBreak2(string s, vector<string>& wordDict)
{
	unordered_map<string, vector<string>> m;
	return helper(s, wordDict, m);
}
```
runtime: 69.58%  
memory usage: 90.91%  

# Link
[leetcode-140](https://leetcode.com/problems/word-break-ii/)  
[solution](https://www.cnblogs.com/grandyang/p/4576240.html)  
