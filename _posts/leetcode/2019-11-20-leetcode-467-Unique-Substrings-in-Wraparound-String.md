---
layout: post
title: "Leetcode467-Unique Substrings in Wraparound String"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-20 20:29:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Consider the string ``s`` to be the infinite wraparound string of "abcdefghijklmnopqrstuvwxyz", so ``s`` will look like this: "...zabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcd....".

Now we have another string ``p``. Your job is to find out how many unique non-empty substrings of ``p`` are present in ``s``. In particular, your input is the string ``p ``and you need to output the number of different non-empty substrings of ``p`` in the string ``s``.

### Note:
``p`` consists of only lowercase English letters and the size of p might be over ``10000``.

### Example 1:
```c++
Input: "a"
Output: 1

Explanation: Only the substring "a" of string "a" is in the string s.
```
### Example 2:
```c++
Input: "cac"
Output: 2
Explanation: There are two substrings "a", "c" of string "cac" in the string s.
```
### Example 3:
```c++
Input: "zab"
Output: 6
Explanation: There are six substrings "z", "a", "b", "za", "ab", "zab" of string "zab" in the string s.
```

# 解法
这题很明显要用动态规划，因为当前位置字符p[i]能否组成在s中的子字符串和前面在s中字符串的长度有关，适合状态转移方程来解。   
首先想了用动态数组dp[i]表示在i位置最长连续子字符串的长度，dp[i]很明显和dp[i-1]有关，如果dp[i]== dp[i - 1] + 1 || dp[i - 1] - dp[i] == 25，则这两个还是连续的，dp[i] = dp[i - 1] + 1。同时需要根据dp[i]往回遍历对应长度内所有以p[i]结尾的字符串看是否出现过，所以需要一个哈希表存储已经出现的子字符串，防止重复计算。这个想法思想是可行的，但TLE，so sad。  
大佬的解法是把动态数组定义为vector<int> dp(26, 0)。 dp[i]表示'a' + i对应字符结尾的最长连续子字符串的长度。假设连续字符串为"abcd"，那么以'd'结尾的在s中的子字符串为"d", "cd", "bcd", "abcd"，这样只要求出所有已26个字符结尾的最长连续子字符串的长度累加就是结果。 
```c++
int findSubstringInWraproundString(string p)
{
	vector<int> cnt(26, 0);
	int n = p.size(), len = 0;
	for (int i = 0; i < n; ++i)
	{
		if (i > 0 && (p[i] - p[i - 1] == 1 || p[i - 1] - p[i] == 25)){
			++len;
		}
		else {
			len = 1;
		}
		cnt[p[i] - 'a'] = max(cnt[p[i] - 'a'], len);
	}
	return accumulate(cnt.begin(), cnt.end(), 0);
}
```
time complexity: O(n)  
space compelxity: O(1)  
runtime: 51.46%  
memory usage:  85.71%  

# Link
[leetcode-467](https://leetcode.com/problems/unique-substrings-in-wraparound-string/)  
[solution](https://www.cnblogs.com/grandyang/p/6143071.html)  