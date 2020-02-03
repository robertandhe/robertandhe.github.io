---
layout: post
title: "Leetcode995-minimum number of K consecutive bit flips"
subtitle: 'Leetcode-greedy, sliding window'
date:       2019-11-01 15:59:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - greedy
  - sliding window
---

# 问题描述
In an array ``A ``containing only 0s and 1s, a ``K``-bit flip consists of choosing a (contiguous) subarray of length ``K`` and simultaneously changing every 0 in the subarray to 1, and every 1 in the subarray to 0.

Return the minimum number of ``K``-bit flips required so that there is no 0 in the array.  If it is not possible, return ``-1``.
### Example 1:
```c++
Input: A = [0,1,0], K = 1
Output: 2
Explanation: Flip A[0], then flip A[2].
```
### Example 2:
```c++
Input: A = [1,1,0], K = 2
Output: -1
Explanation: No matter how we flip subarrays of size 2, we can't make the array become [1,1,1].
```
### Example 3:
```c++
Input: A = [0,0,0,1,0,1,1,0], K = 3
Output: 3
Explanation:
Flip A[0],A[1],A[2]: A becomes [1,1,1,1,0,1,1,0]
Flip A[4],A[5],A[6]: A becomes [1,1,1,1,1,0,0,0]
Flip A[5],A[6],A[7]: A becomes [1,1,1,1,1,1,1,1]
```
### Note:
1. ``1 <= A.length <= 30000``
2. ``1 <= K <= A.length``


# 解法
## 解法 1
naive approach。遇到0，就翻转后续K个，遇到1就继续。最后判断末尾K个里面有没有0。但是无法通过OJ。
```c++
int minKBitFlips(vector<int>& A, int K)
{
	int res = 0, n = A.size();
	for (int i = 0; i <= n - K; ++i)
	{
		if (A[i] == 1)
			continue;
		++res;
		int j = i;
		while (j < i + K && A[j] == 0)
			A[j++] = 1;
		int t = j;
		for (; j < i + K; ++j)
			A[j] = A[j] == 0 ? 1 : 0;
		i = t - 1;
	}
	for (int i = n - K; i < n; ++i)
	{
		if (A[i] == 0)
			return -1;
	}
	return res;
}
```

## 解法 2
滑动窗口法。因为要连续翻转K个，用一个和原始数组等长的vector isFlipped存储开始翻转位置。当前位置会受到前面K长度范围内所有翻转影响。用变量curFlipped表示当前位置是否翻转，每遇到一个开始翻转位置，curFlipped与1异或，异或偶数次curFlipped表示当前位置没有被翻转（其实是翻转了偶数次）。那么如何判断当前位置需不需要开始翻转，有两种情况需要开始翻转：
- curFlipped为1，当前A[i]==1，因为curFlipped为1表示当前位置被前面的翻转影响导致翻转，但同时当前A[i]==1,翻转后为0，需要翻转；
- curFlipped为0，当前A[i]==0，前面翻转在当前位置的总效果为不翻转，当前位置也等于0，需要翻转。
总的来说就是curFlipped % 2 == A[i]时要开始做翻转。

同时curFlipped只受到K长度内开始翻转影响，超出时curFplipped应该与isFlipped[i-K]异或，其实就是去掉前面K位置翻转影响，如果前面K位置isFlipped[i-K]为1，表示翻转了，那就要翻回去；如果isFplipped[i-K]为0，表示前面没有翻转，那就啥也不干，与0异或。
```c++
int minKBitFlips(vector<int>& A, int K)
{
	int res = 0, n = A.size(), curFlipped = 0;
	vector<int> isFlipped(n, 0);
	for (int i = 0; i < n; ++i)
	{
		if (i >= K)
			curFlipped ^= isFlipped[i - K];
		if (curFlipped % 2 == A[i]){
			if (i + K > n)
				return -1;
			++res;
			curFlipped ^= 1;
			isFlipped[i] = 1;
		}
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(n)  
runtime: 75.62%  
memory usage: 100.00%  

## 解法 3
解法2需要一个和原始数组等长的向量，其实当前curFlipped只与前面K长度内翻转有关，因此这里建立一个队列来存储之前开始翻转的位置。如果当前位置i大于队列开始位置+K,表示队列开始位置对应的翻转没法影响当前位置，弹出。同时和解法2一样满足开始翻转条件时，当前位置i推入队列。前面翻转在当前位置的总效果可用队列长度余2表示，如果余数为0，表示偶数次翻转，总效果为不翻转，反之亦然。
```c++
int minKBitFlips(vector<int>& A, int K)
{
	int res = 0, n = A.size();
	queue<int> q;
	for (int i = 0; i < n; ++i)
	{
		if (!q.empty() && i >= (q.front() + K))
			q.pop();
		if (q.size() % 2 == A[i]){
			if (i + K > n)
				return -1;
			q.push(i);
			++res;
		}
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 75.62%  
memory usage: 100.00%  

## 解法 4
这里直接在原始数组上修改，原始数组只有0和1，当开始翻转时，加2，到时候除2的商就表示翻转了没有。其余思想和解法2，3相同。
```c++
int minKBitFlips(vector<int>& A, int K)
{
	int res = 0, n = A.size(), flipped = 0;
	for (int i = 0; i < n; ++i)
	{
		if (i >= K)
			flipped -= A[i - K] / 2;
		if (flipped % 2 == A[i]){
			if (i + K > n)
				return -1;
			A[i] += 2;
			++flipped;
			++res;
		}
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 90.29%  
memory usage: 100.00%  

# link
[leetcode-995](https://leetcode.com/problems/minimum-number-of-k-consecutive-bit-flips/)  
[solution 2,3,4](https://www.cnblogs.com/grandyang/p/10793594.html)
