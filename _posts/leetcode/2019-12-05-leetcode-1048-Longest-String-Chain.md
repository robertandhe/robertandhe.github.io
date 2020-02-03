---
layout: post
title: "Leetcode1048 - Longest String Chain"
subtitle: 'Leetcode1048-dynamic programming'
date:       2019-12-05 11:11:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given a list of words, each word consists of English lowercase letters.

Let's say ``word1`` is a predecessor of ``word2`` if and only if we can add exactly one letter anywhere in ``word1`` to make it equal to ``word2``.  For example, ``"abc"`` is a predecessor of ``"abac"``.

A word chain is a sequence of words [``word_1, word_2, ..., word_k``] with ``k >= 1``, where ``word_1`` is a predecessor of ``word_2``, ``word_2`` is a predecessor of ``word_3``, and so on.

Return the longest possible length of a word chain with words chosen from the given list of ``words``.

### Example 1:
```c++
Input: ["a","b","ba","bca","bda","bdca"]
Output: 4
Explanation: one of the longest word chain is "a","ba","bda","bdca".
```
### Note:
1. ``1 <= words.length <= 1000``
2. ``1 <= words[i].length <= 16``
3. ``words[i]`` only consists of English lowercase letters.

# 解法
&emsp;题目给出一些字符串，要找出其中最长的字符串链条，两个字符串可以链接的意思是后一个字符串比前一个字符串多一个字符，其余字符的相对位置均相同。  
&emsp;用动态规划来解，动态数组dp[i]表示包括i位置字符串的最长的链长度。状态转移方程也挺常规，从i往前遍历，如果words[i]比words[j]长1，则可以进行比较步骤，比较成功则可以链起来，链起来的话dp[i] = max(dp[i], dp[j] + 1)；如果长度不是相差1，则直接继续往前。  
&emsp;那么当两个字符串长度相差1时，如何判断它们可以链接。从末尾同时往前遍历，如果到某个位置words[i][t1] != words[j][t2], t1减1，向前调整1个位置，再继续往前，一直到开头所有位置字符都相等才能算可以链接。  
&emsp;因为dp[i]表示包括i位置字符串的最长的链长度，所以res应该在i位置更新完后更新一次， res=max(res, dp[i])。
```c++
int longestStrChain(vector<string>& words)
{
	int res = 1, n = words.size();
	sort(words.begin(), words.end(), [](string& s1, string& s2) {return s1.length() < s2.length();});
	vector<int> dp(n, 1);
	for (int i = 1; i < n; ++i)
	{
		for (int j = i - 1; j >= 0; --j)
		{
			if (words[i].length() - words[j].length() != 1)
				continue;
			string s1 = words[i], s2 = words[j];
			bool flag = false;
			int t1 = s1.size() - 1, t2 = s2.size() - 1;
			while (t1 >= 0 && t2 >= 0 && s1[t1] == s2[t2]){
				--t1;
				--t2;
			}
			--t1;
			while (t1 >= 0 && s1[t1] == s2[t2]){
				--t1;
				--t2;
			}
			if (t1 < 0)
				dp[i] = max(dp[i], dp[j] + 1);
			}
			res = max(res, dp[i]);
	}
	return res;
}
```
time complexity: O(n^2)  
space complexity: O(n)  
runtime: 79.76%  
memory usage: 100.00%  

# Link
[leetcode-1048](https://leetcode.com/problems/longest-string-chain/)