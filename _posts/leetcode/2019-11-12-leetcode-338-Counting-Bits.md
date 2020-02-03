---
layout: post
title: "Leetcode338-Counting Bits"
subtitle: 'Leetcode-stack'
date:       2019-11-12 14:41:00
author: "mudux"
header-style: text
catalog: true
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - bitset
---

# 问题描述
Given a non negative integer number ``num``. For every numbers ``i`` in the range ``0 ≤ i ≤ num`` calculate the number of 1's in their binary representation and return them as an array.
### Example 1:
```c++
Input: 2
Output: [0,1,1]
```
### Example 2:
```c++
Input: 5
Output: [0,1,1,2,1,2]
```
### Follow up:
- It is very easy to come up with a solution with run time ``O(n*sizeof(integer))``. But can you do it in linear time ``O(n)`` /possibly in a single pass?
- Space complexity should be ``O(n)``.
- Can you do it like a boss? Do it without using any builtin function like ``__builtin_popcount`` in c++ or in any other language.

# 解法
## 解法 1
题目要求统计[0, num]范围内每个数字二进制表示中1的个数，返回数组。那么先上naive approach :)。
```c++
vector<int> countBits(int num)
{ 
	vector<int> res;
	for (int i = 0; i <= num; ++i)
	{
		int t = i, cnt = 0;
		while (t){
			if (t & 1)
				++cnt;
			t = t >> 1;
		}
		res.push_back(cnt);
	}
	return res;
}
```
time complexity: O(n*sizeof(interger))  
space complexity: O(n)  
runtime: 56.31%  
memory usage: 36.59%  

## 解法 2
该上动态规划了，根据题目hint也可知应该把数字分区间[2, 3), [4, 8), [8, 15)。因为到二进制指数是1的个数为1，而在当前指数到下个指数区间内1的个数刚好等于上个区间1的个数加最开始的1. perfect!
```c++
vector<int> countBits(int num)
{
	vector<int> res(num + 1, 0);
	if (num == 0)
		return {0};
	else if (num == 1)
		return {0, 1};
	res[0] = 0;
	res[1] = 1;
	int i = 2;
	for (i = 2; 2 * i - 1 <= num; i *= 2)
	{
		res[i] = 1;
		for (int j = i + 1; j <= 2 * i - 1; ++j)
		{
			res[j] = res[j - i] + 1;
		}
	}
	if (i <= num){
		res[i] = 1;
		for (int j = i + 1; j <= num; ++j)
		{
			res[j] = res[j - i] + 1;
		}
	}
	return res;
}
```
timecomplexity: O(n)  
space complexity: O(n)  
runtime: 56.60%  
memory usage: 100.00%  

## 解法 3
解法2略显繁琐，简化。
```c++
vector<int> countBits(int num)
{
	vector<int> res(num + 1, 0);
	int i = 1;
	for (i = 1; i <= num; i *= 2)
	{
		res[i] = 1;
		for (int j = i + 1; j <= 2 * i - 1 && j <= num; ++j)
		{
			res[j] = res[j - i] + 1;
		}
	}
	return res;
}
```
timecomplexity: O(n)  
space complexity: O(n)  
runtime: 56.60%  
memory usage: 75.61%  

## 解法 4
使用内置的bitset
```c++
vector<int> countBits(int num)
{
	vector<int> res;
	for (int i = 0; i <= num; ++i)
		res.push_back(bitset<32>(i).count());
	return res;
}
```
runtime: 56.60%  
memory usage: 36.59%  

## 解法 5
根据hint利用奇偶性。因为当前数字i为偶数时，它应该是前面某个数字i/2左移一位，1的个数不变。当前数字i为奇数时，它应该是前面某个数字i/2的1个数加1.
```c++
vector<int> countBits(int num)
{
	vector<int> res(num + 1, 0);
	for (int i = 0; i <= num; ++i)
	{
		if (i % 2)
			res[i] = res[i / 2] + 1;
		else 
			res[i] = res[i / 2];
	}
	return res;
}
```
timecomplexity: O(n)  
space complexity: O(n)  
runtime: 93.92%  
memory usage: 92.68%  

## 解法 6
挺精妙。利用i&(i - 1)。

|  i   | binary | #'1' | i&(i-1) |
| :--: | :----: | :--: | :-----: |
|  0   |  0000  |  0   |         |
|  1   |  0001  |  1   |  0000   |
|  2   |  0010  |  1   |  0000   |
|  3   |  0011  |  2   |  0010   |
|  4   |  0100  |  1   |  0000   |
|  5   |  0101  |  2   |  0100   |
|  6   |  0110  |  2   |  0100   |
|  7   |  0111  |  3   |  0110   |
|  8   |  1000  |  1   |  0000   |
|  9   |  1001  |  2   |  1000   |
|  10  |  1010  |  2   |  1000   |
|  11  |  1011  |  3   |  1010   |
|  12  |  1100  |  2   |  1000   |
|  13  |  1101  |  3   |  1100   |
|  14  |  1110  |  3   |  1100   |
|  15  |  1111  |  4   |  1110   |

可以发现每个i值都是 i&(i-1) 对应的值加1。
```c++
vector<int> countBits(int num) {
    vector<int> res(num + 1, 0);
    for (int i = 1; i <= num; ++i) {
        res[i] = res[i & (i - 1)] + 1;
    }
    return res;
}
```
timecomplexity: O(n)  
space complexity: O(n)  
runtime: 93.92%  
memory usage:82.93%  

# Link
[leetcode-338](https://leetcode.com/problems/counting-bits/)  
[solution 4-6](https://www.cnblogs.com/grandyang/p/5294255.html)