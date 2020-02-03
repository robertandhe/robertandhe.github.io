---
layout: post
title: "Leetcode846 - Hand of Straights"
subtitle: 'Leetcode - ordered map'
date:       2019-12-23 19:27:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - ordered map
---

## 问题描述
Alice has a hand of cards, given as an array of integers.

Now she wants to rearrange the cards into groups so that each group is size ``W``, and consists of ``W`` consecutive cards.

Return ``true`` if and only if she can.
#### Example 1
```c++
Input: hand = [1,2,3,6,2,3,4,7,8], W = 3
Output: true
Explanation: Alice's hand can be rearranged as [1,2,3],[2,3,4],[6,7,8].
```
#### Example 2
```c++
Input: hand = [1,2,3,4,5], W = 4
Output: false
Explanation: Alice's hand can't be rearranged into groups of 4.
```
#### Note:
1. ``1 <= hand.length <= 10000``
2. ``0 <= hand[i] <= 10^9``
3. ``1 <= W <= hand.length``

## 解法
### 解法 1
和weekly contest 168第二道题很相似，解法也基本相同。
```c++
bool isNStraightHand(vector<int>& hand, int W) 
{
	int n = hand.size();
	if (n % W != 0)
		return false;
	map<int, int> m;
	for (int& card : hand)
		m[card]++;
	for (auto& a : m)
	{
		if (a.second == 0)
			continue;
		int num = a.first;
		int cnt = a.second;
		for (int i = 1; i < W; ++i)
		{
			if (m.count(num + i) && m[num + i] >= cnt){
				m[num + i] -= cnt;
			}
			else 
				return false;
		}
	}
	return true;
}
```
time complexity: O(n * W)  
space complexity: O(n)  
runtime: 51.08%  
memory usage: 66.67%  

### 解法 2
有序表不用担心键值不存在，还能进行时间优化。
```c++
bool isNStraightHand2(vector<int>& hand, int W) 
{
	int n = hand.size();
	if (n % W != 0)
		return false;
	map<int, int> m;
	for (int& card : hand)
		m[card]++;
	for (auto& a : m)
	{
		if (a.second == 0)
			continue;
		int num = a.first;
		int cnt = a.second;
		for (int i = 1; i < W; ++i)
		{
			if ((m[num + i] -= cnt) < 0)
				return false;
		}
	}
	return true;
}
```
time complexity: O(n * W)  
space complexity: O(n)  
runtime: 83.55%  
memory usage: 66.67%  

## Link
[leetcode-846](https://leetcode.com/problems/hand-of-straights/)  