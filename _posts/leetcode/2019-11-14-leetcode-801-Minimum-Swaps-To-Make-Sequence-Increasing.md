---
layout: post
title: "Leetcode801-Minimum Swaps To Make Sequences Inreasing"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-14 15:39:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
We have two integer sequences ``A`` and ``B`` of the same non-zero length.

We are allowed to swap elements ``A[i]`` and ``B[i]``.  Note that both elements are in the same index position in their respective sequences.

At the end of some number of swaps, ``A`` and ``B`` are both strictly increasing.  (A sequence is strictly increasing if and only if ``A[0] < A[1] < A[2] < ... < A[A.length - 1]``.)

Given A and B, return the minimum number of swaps to make both sequences strictly increasing.  It is guaranteed that the given input always makes it possible.
### Example:
```c++
Input: A = [1,3,5,4], B = [1,2,3,7]
Output: 1
Explanation: 
Swap A[3] and B[3].  Then the sequences are:
A = [1, 3, 5, 7] and B = [1, 2, 3, 4]
which are both strictly increasing.
```
### Note:
- ``A, B`` are arrays with the same length, and that length will be in the range ``[1, 1000]``.
- ``A[i], B[i]`` are integer values in the range ``[0, 2000]``.

# 解法
## 解法 1
这题感觉就是动态规划，不过没搞出来。感觉当前位置交换后也会影响后面的交换，所以挺复杂的。看了大佬的解答才懂。  
动态规划首先就是定义dp数组，在这道题中如果把dp数组定义为dp[i]表示[0, i]范围内使子数组严格递增的最小交换次数，那么状态转移方程会比较难写，因为
没有分开内部的关联，当前位置i是否交换，只取决于和前面一位是否严格递增，而前面一位又有交换和不交换两种选择，和更前面的有关。这道题中我们需要维护两种状态，swap和noSwap，那么swap[i]表示范围[0, i]的子数组同时严格递增且当前位置i需要交换的最小交换次数，noSwap[i]表示[0, i]内的子数组严格递增且当前位置i不交换的最小交换次数。swap[0]初始化为1，noSwap[0]初始化为0。  
接下来解决状态转移方程，因为题目限定一定有解，所以只有两种情况。
- A[i] > A[i - 1] && B[i] > B[i - 1];
- A[i] > B[i - 1] && B[i] > A[i - 1]。
注意上述两种情况可能同时满足，此时要做的就是取最小值。如B={1, 3, 5, 7}, A={2, 4, 6, 8}。



1. 假设当前位置不用交换，当前位置i的数字已经分别大于前一个位置i-1。更新动态数组，因为swap[i]限定了i位置交换，那么i-1位置也要交换，swap[i] = swap[i-1]（或者swap[i] = min(swap[i], swap[i-1]+1)）;如果i-1位置没有交换，i位置也就不用交换了，noSwap[i] = noSwap[i - 1]（或者(noSwap[i]= min(noSwap[i], noSwap[i - 1]))）。
2. 假设当前位置需要交换，则swap[i] = min(swap[i], noSwap[i - 1] + 1), noSwap[i] = min(noSwap[i], Swap[i - 1])。

```c++
int minSwap(vector<int>& A, vector<int>& B)
{
	int n = A.size();
	vector<int> Swap(n, n), noSwap(n, n);
	Swap[0] = 1;
	noSwap[0] = 0;
	for (int i = 1; i < n; ++i)
	{
		if (A[i] > A[i - 1] && B[i] > B[i - 1]){
			Swap[i] = min(Swap[i], Swap[i - 1] + 1);
			noSwap[i] = min(noSwap[i], noSwap[i - 1]);
		}
		if (A[i] > B[i - 1] && B[i] > A[i - 1]) {
			Swap[i] = min(Swap[i], noSwap[i - 1] + 1);
			noSwap[i] = min(noSwap[i], Swap[i - 1]);
		}
	}
	return min(Swap[n - 1], noSwap[n - 1]);
}
```
注意这里代码中是两个if，原因如上所述，这两种情况可能同时满足，此时就要取最小的。

```c++
int minSwap(vector<int>& A, vector<int>& B)
{
	int n = A.size();
	vector<int> Swap(n, n), noSwap(n, n);
	Swap[0] = 1;
	noSwap[0] = 0;
	for (int i = 1; i < n; ++i)
	{
		if (A[i] > A[i - 1] && B[i] > B[i - 1]){
			Swap[i] = Swap[i - 1] + 1;
			noSwap[i] = noSwap[i - 1];
		}
		if (A[i] > B[i - 1] && B[i] > A[i - 1]) {
			Swap[i] = min(Swap[i], noSwap[i - 1] + 1);
			noSwap[i] = min(noSwap[i], Swap[i - 1]);
		}
	}
	return min(Swap[n - 1], noSwap[n - 1]);
}
```
两个代码是一样的，因为第一个if一定会减小Swap[i]和noSwap[i]。  
time complexity: O(n)  
space complexity: O(n)  
runtime: 75.63%  
memory usage: 50.00%  

## 解法 2
空间优化。i位置至于i-1位置有关，用两个变量存储即可。
```c++
int minSwap3(vector<int>& A, vector<int>& B)
{
	int n1 = 0, s1 = 1;
	for (int i = 1; i < A.size(); ++i)
	{
		int n2 = INT_MAX, s2 = INT_MAX;
		if (A[i] > A[i - 1] && B[i] > B[i - 1]){
			s2 = s1 + 1;
			n2 = n1;
		}
		if (A[i] > B[i - 1] && B[i] > A[i - 1]){
			s2 = min(s2, n1 + 1);
			n2 = min(n2, s1);
		}
		n1 = n2;
		s1 = s2;
	}
	return min(n1, s1);
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 97.74%  
memory usage: 90.00%  

# Link
[leetcode-801](https://leetcode.com/problems/minimum-swaps-to-make-sequences-increasing/)  
[solution](https://www.cnblogs.com/grandyang/p/9311385.html)  