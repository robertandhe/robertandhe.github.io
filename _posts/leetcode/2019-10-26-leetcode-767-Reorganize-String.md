---
layout: post
title: "Leetcode767-Reorganize String"
subtitle: 'Leetcode767-greedy,hash-table,heap'
date:       2019-10-26 11:41:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
  - priority-queue
---

# 问题描述
Given a string ``S``, check if the letters can be rearranged so that two characters that are adjacent to each other are not the same.

If possible, output any possible result.  If not possible, return the empty string.
### Example 1:
```c++
Input: S = "aab"
Output: "aba"
```
### Example 2:
```c++
Input: S = "aaab"
Output: ""
```
### Note:
- ``S`` will consist of lowercase letters and have length in range ``[1, 500]``.

# 解法
## 解法1
题目要求重组字符串是相邻字符不相同。很明显应该用hash table统计字符出现次数，在重组时按照出现次数最多的字符优先排的原则来重组。但是hash table按照value排序略显复杂，所以借助priority queue来排序。这里有个技巧，每次抽出前两个出现次数最多的字符，因为这样取产生的字符串肯定是符合题目要求的，如果每次只取一个次数最多的字符，可能出现重复抽取。然后在构造hash table时如果某个字符出现次数超过原始字符串长度一半则返回空字符串。
```c++
string reorganizeString(string S)
{
	string res = "";
	unordered_map<char, int> m;
	priority_queue<pair<int, char>> q;
	for (char&c : S)
		++m[c];
	for (auto&a : m){
		if (a.second > (S.size() + 1) / 2 )
			return "";
		q.push({a.second, a.first});
	}
	while (q.size() >= 2){
		auto t1 = q.top();
		q.pop();
		auto t2 = q.top();
		q.pop();
		res += t1.second;
		res += t2.second;
		if (--t1.first > 0)
			q.push(t1);
		if (--t2.first > 0)
			q.push(t2);
	}
	if (q.size()){
		res += q.top().second;
	}
	return res;
}
```
时间复杂度：O(n^2log(n))  
空间复杂度：O(n)  
runtime: 70.55%  
memory usage: 100.00%  
## 解法2
因为字符串全是小写字母，可用长度为26的vector来存储，但在重组时需要知道字符和字符次数，这里用了技巧，每出现一次加100，最后加上字符索引，使一个数字同时编码次数和对应的字符（只有26个字符，索引不会超100）。解码是除100就是次数，除100取余加``'a'``就是字符。  
vector按从小到大排序，先放出现次数少的，第二个技巧就是字符串第二个位置开始放，因为出现次数最多的字符肯定放0位置。如果idx超出长度，则置为0. 
```c++
string reorganizeString(string S)
{
	vector<int> vec(26, 0);
	for (char& c : S)
		vec[c - 'a'] += 100;
	for (int i = 0; i < 26; ++i)
		vec[i] += i;
	sort(vec.begin(), vec.end());
	int n = S.size(), idx = 1;
	for (int& a : vec)
	{
		int t = a / 100;
		if (t > (n + 1) / 2)
			return "";
		char ch = 'a' + a % 100;
		for (int i = 0; i < t; ++i)
		{
			if (idx >= n)
				idx = 0;
			S[idx] = ch;
			idx += 2;
		}
	}
	return S;
}
```
时间复杂度：O(n)  
空间复杂度：O(1)  
runtime: 70.46%  
memory usage: 100.00%  


## 解法3
和解法2一样，只是vector排序时按照从大到小排，idx从0开始，到末尾时置为1.
```c++
string reorganizeString3(string S)
{
	vector<int> vec(26, 0);
	for (char& c : S)
		vec[c - 'a'] += 100;
	for (int i = 0; i < 26; ++i)
		vec[i] += i;
	sort(vec.begin(), vec.end(), greater<int>());
	int n = S.size(), idx = 0;
	for (int& a : vec)
	{
		int t = a / 100;
		if (t > (n + 1) / 2)
			return "";
		char ch = 'a' + a % 100;
		for (int i = 0; i < t; ++i)
		{
			if (idx >= n)
				idx = 1;
			S[idx] = ch;
			idx += 2;
		}
	}
	return S;
}
```
时间复杂度：O(n)  
空间复杂度：O(1)  
runtime: 100.00%  
memory usage: 100.00% 
# link
[leetcode-767](https://leetcode.com/problems/reorganize-string/)  
[solution](https://www.cnblogs.com/grandyang/p/8799483.html)