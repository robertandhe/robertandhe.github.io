---
layout: post
title: "Leetcode967 - Numbers With Same Consecutive Differences"
subtitle: 'Leetcode-dynamic programming, recursion'
date:       2019-12-02 21:24:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - recursion
---

# 问题描述
Return all **non-negative** integers of length ``N`` such that the absolute difference between every two consecutive digits is ``K``.

Note that **every** number in the answer **must not** have leading zeros **except** for the number ``0`` itself. For example, ``01`` has one leading zero and is invalid, but ``0`` is valid.

You may return the answer in any order.
### Example 1:
```c++
Input: N = 3, K = 7
Output: [181,292,707,818,929]
Explanation: Note that 070 is not a valid number, because it has leading zeroes.
```
### Example 2:
```c++
Input: N = 2, K = 1
Output: [10,12,21,23,32,34,43,45,54,56,65,67,76,78,87,89,98]
```
### Note:
1. ``1 <= N <= 9``
2. ``0 <= K <= 9``

# 解法
## 解法 1
&emsp;这题要求出所有数字内前后两位差值绝对值为K且长度为N的数组，数字不能有前导0，0除外。  
&emsp;首先用递归求解，因为递归时可以用向量存储前面那一个位置数值，接下来的数字和它的差的绝对值必须为K。找到差值为K的推入向量，剩余长度减1，进入下层递归即可。如果数组长度达到N，把向量转换为数字存入res。  
&emsp;这里可能有重复数组，所以递归完要先用set存一下去重，再转为vector返回。
```c++
void helper(vector<int>& arr, int N, int& K, vector<int>& res)
{
	if (N == 0){
		int sum = 0;
		for (int& a : arr)
			sum = 10 * sum + a;
		res.push_back(sum);
		return;
	}
	if (arr[0] == 0)
		return;
	if (arr.back() + K < 10){
		arr.push_back(arr.back() + K);
		helper(arr, N - 1, K, res);
		arr.pop_back();
	}
	if (arr.back() - K >= 0){
		arr.push_back(arr.back() - K);
		helper(arr, N - 1, K, res);
		arr.pop_back();
	}
}

vector<int> numsSameConsecDiff(int N, int K) 
{
	vector<int> res;
	vector<int> arr;
	for (int i = 0; i < 10; ++i)
	{
		arr.push_back(i);
		helper(arr, N - 1, K, res);
		arr.pop_back();
	}
		
	set<int> st(res.begin(), res.end());
	return vector<int>(st.begin(), st.end());
}
```
runtime: 6.53%  
memory usage: 16.67%  
&emsp;这里还能优化，用一个数存储当前数即可，末尾数字用它对10取余即可得到，重复数字只能在K==0时出现，区分对待一下即可。

## 解法 2
&emsp;动态规划，用res向量存储当前数字，因为数字长度为N，按道理应循环N次，当把res初始化为0-9时用去一位，只用循环N-1次。  
&emsp;状态转移就是先申请一个新的向量arr，对res中每个数字，对10取余得到末尾数值，再用末尾数字加减K看是否在0-9范围内，如果在就可加到最后位，为了防止出现重复数字，当K==0时，减K就不做了。先导位为0时，加减都不做，直接continue。
```c++
vector<int> numsSameConsecDiff(int N, int K) 
{
	vector<int> res = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
	for (int i = 1; i < N; ++i)
	{
		vector<int> arr;
		for (int& num : res)
		{
			if (!num % 10)
				continue;
			if (num % 10 + K < 10){
				arr.push_back(num * 10 + num % 10 + K);
			}
			if (num % 10 - K >= 0 && K != 0)
				arr.push_back(num * 10 + num % 10 - K);
		}
		res = arr;
	}
	return res;
}
```
runtime:  97.43%  
memory usage: 83.33%  

# Link
[leetcode-967](https://leetcode.com/problems/numbers-with-same-consecutive-differences/)
