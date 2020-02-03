---
layout: post
title: "Leetcode975 - Odd Even Jump"
subtitle: 'Leetcode - ordered map, dynamic programming'
date:       2019-12-23 22:29:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - ordered map
  - dynamic programming
---

## 问题描述
You are given an integer array ``A``.  From some starting index, you can make a series of jumps.  The (1st, 3rd, 5th, ...) jumps in the series are called *odd numbered jumps*, and the (2nd, 4th, 6th, ...) jumps in the series are called *even numbered jumps*.

You may from index ``i`` jump forward to index ``j`` (with ``i < j``) in the following way:
- During odd numbered jumps (ie. jumps 1, 3, 5, ...), you jump to the index j such that ``A[i] <= A[j]`` and ``A[j]`` is the smallest possible value.  If there are multiple such indexes j, you can only jump to the **smallest** such index ``j``.
- During even numbered jumps (ie. jumps 2, 4, 6, ...), you jump to the index ``j`` such that ``A[i] >= A[j]`` and ``A[j]`` is the largest possible value.  If there are multiple such indexes j, you can only jump to the **smallest** such index ``j``.
- (It may be the case that for some index ``i``, there are no legal jumps.)

A starting index is good if, starting from that index, you can reach the end of the array (index ``A.length - 1``) by jumping some number of times (possibly 0 or more than once.)

Return the number of good starting indexes.

#### Example 1:
```c++
Input: [10,13,12,14,15]
Output: 2
Explanation: 
From starting index i = 0, we can jump to i = 2 (since A[2] is the smallest among A[1], A[2], A[3], A[4] that is greater or equal to A[0]), then we can't jump any more.
From starting index i = 1 and i = 2, we can jump to i = 3, then we can't jump any more.
From starting index i = 3, we can jump to i = 4, so we've reached the end.
From starting index i = 4, we've reached the end already.
In total, there are 2 different starting indexes (i = 3, i = 4) where we can reach the end with some number of jumps.
```
#### Example 2:
```c++
Input: [2,3,1,1,4]
Output: 3
Explanation: 
From starting index i = 0, we make jumps to i = 1, i = 2, i = 3:

During our 1st jump (odd numbered), we first jump to i = 1 because A[1] is the smallest value in (A[1], A[2], A[3], A[4]) that is greater than or equal to A[0].

During our 2nd jump (even numbered), we jump from i = 1 to i = 2 because A[2] is the largest value in (A[2], A[3], A[4]) that is less than or equal to A[1].  A[3] is also the largest value, but 2 is a smaller index, so we can only jump to i = 2 and not i = 3.

During our 3rd jump (odd numbered), we jump from i = 2 to i = 3 because A[3] is the smallest value in (A[3], A[4]) that is greater than or equal to A[2].

We can't jump from i = 3 to i = 4, so the starting index i = 0 is not good.

In a similar manner, we can deduce that:
From starting index i = 1, we jump to i = 4, so we reach the end.
From starting index i = 2, we jump to i = 3, and then we can't jump anymore.
From starting index i = 3, we jump to i = 4, so we reach the end.
From starting index i = 4, we are already at the end.
In total, there are 3 different starting indexes (i = 1, i = 3, i = 4) where we can reach the end with some number of jumps.
```
#### Example 3:
```c++
Input: [5,1,3,4,2]
Output: 3
Explanation: 
We can reach the end from starting indexes 1, 2, and 4.
```
#### Note:
1. ``1 <= A.length <= 20000``
2. ``0 <= A[i] < 100000``

## 解法
&emsp;这道题给出一种跳跃方式，在奇数步跳跃时在当前数后面位置选中大于等于当前数的最小数当作目标位置，在偶数步是在当前数后面位置选中小于等于当前数的最大数当作目标位置，如果有多个位置满足要求，选取索引最小的位置，返回能跳到末尾的起跳点个数。  
&emsp;如果从前往后遍历无法知道后面那个位置符合要求，所以应该从后往前遍历。那么怎么知道那个位置符合要求，必须同时存储数值和位置，还要有序，就只有用有序图了，ordered_map<int, int>m 键为数值，值为位置。同时跳跃方式有跳低和跳高两种，所有要两个动态布尔数组来表示，higher[i]为true表示从i位置跳高能够到达末尾，lower[i]同理。  
&emsp;那么状态转移方程如何求。假设在当前位置时跳高，在后面位置选取大于等于它的位置，用m.lower_bound(A[i]）就能找到，用auto hi = m.lower_bound(A[i])表示下个位置，因为这步跳高了，下步必须跳低，能够到达末尾要看lower[hi->second]是否为true了。假设在当前位置跳低，在后面位置选取小于等于它的位置，注意这里用auto lo = m.upper_bound(A[i])，原因后面解释，因为这步跳低，下一步就是跳高，这个位置跳低能够到达末尾位置要看higher[(--lo)->second]。最后第一步必须跳高，以当前位置为起点能够到达末尾就要看higher[i]是否为true。  
&emsp;举例来说：
We need to jump higher and lower alternately to the end.

Take [5,1,3,4,2] as example.

If we start at 2,
we can jump either higher first or lower first to the end,
because we are already at the end.
higher(2) = true
lower(2) = true

If we start at 4,
we can't jump higher, higher(4) = false
we can jump lower to 2, lower(4) = higher(2) = true

If we start at 3,
we can jump higher to 4, higher(3) = lower(4) = true
we can jump lower to 2, lower(3) = higher(2) = true

If we start at 1,
we can jump higher to 2, higher(1) = lower(2) = true
we can't jump lower, lower(1) = false

If we start at 5,
we can't jump higher, higher(5) = false
we can jump lower to 4, lower(5) = higher(4) = false

&emsp;再解释为啥auto lo = m.upper_bound(A[i])，以example 1为例，A=[10,13,12,14,15]。到倒数第二个14时，此时没有小于等于它的数，没法跳低，如果upper_bound知道begin，说明之前没有小于等于它的数，就没法跳了。

```c++
int oddEvenJumps(vector<int>& A) 
{
	int res = 1, n = A.size(); 
	vector<bool> higher(n), lower(n);
	higher[n - 1] = true;
	lower[n - 1] = true;
	map<int, int> m;
	m[A[n - 1]] = n - 1;
	for (int i = n - 2; i >= 0; --i)
	{
		auto hi = m.lower_bound(A[i]);
		auto lo = m.upper_bound(A[i]);
		if (hi != m.end()){
			higher[i] = lower[hi->second];
		}
		if (lo != m.begin()){
			lower[i] = higher[(--lo)->second];
		}
		if (higher[i])
			++res;
		m[A[i]] = i;
	}
	return res;
}
```

## Link
[leetcode-975](https://leetcode.com/problems/odd-even-jump/)  
[solution](https://leetcode.com/problems/odd-even-jump/discuss/217981/JavaC%2B%2BPython-DP-idea-Using-TreeMap-or-Stack)  