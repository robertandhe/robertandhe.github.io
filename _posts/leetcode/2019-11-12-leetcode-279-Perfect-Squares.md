---
layout: post
title: "Leetcode279-Perfect Squares"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-12 19:02:00
author: "mudux"
header-style: text
catalog: false
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given a positive integer n, find the least number of perfect square numbers (for example, ``1, 4, 9, 16, ...``) which sum to ``n``.
### Example 1:
```c++
Input: n = 12
Output: 3 
Explanation: 12 = 4 + 4 + 4.
```
### Example 2:
```c++
Input: n = 13
Output: 2
Explanation: 13 = 4 + 9.
```

# 解法
## 解法 1
这题一看就和之前这道[Coin Change](https://hexinlin.top/2019/11/11/leetcode-322-Coin-Change/)如出一辙。这里coins就是完全平方数，自己生成即可。
```c++
int numSquares(int n)
{
	vector<int> dp(n + 1, n + 1);
	dp[0] = 0;
	vector<int> squares;
	for (int t = 1; t * t <= n; ++t)
		squares.push_back(t * t);
	for (int i = 0; i <= n; ++i)
	{
		for (int j = 0; j < squares.size(); ++j)
		{
			if (i >= squares[j]){
				dp[i] = min(dp[i], dp[i - squares[j]] + 1);
			}
		}
	}
	return dp[n];
}
```
time complexity: O(n^2)  
space complexity: O(n)  
runtime: 18.12%  
memory usage: 30.77%  

## 解法 2
简化解法1，因为这里完全平方数是有规律的，没必要用数组存储。
```c++
int numSquares(int n)
{
	vector<int> dp(n + 1, n + 1);
	dp[0] = 0;
	for (int i = 0; i <= n; ++i)
	{	
		for (int t = 1; t * t <= i; ++t)
			dp[i] = min(dp[i], dp[i - t * t] + 1);
	}
	return dp[n];
}
```
time complexity: O(n^2)  
space complexity: O(n)  
runtime: 62.97%  
memory usage: 76.92%  

## 解法 3
这道题实际上是考察[四平方和定理](https://zh.wikipedia.org/wiki/%E5%9B%9B%E5%B9%B3%E6%96%B9%E5%92%8C%E5%AE%9A%E7%90%86)，即任意正整数最多是4个平方数的和。如果一个数除以8余7，那么肯定是由4个完全平方数组成。如果数字是由1个或者2个完全平方数组成，则可以分解。
```c++
int numSquares(int n) {
    while (n % 4 == 0) n /= 4;
    if (n % 8 == 7) return 4;
    for (int a = 0; a * a <= n; ++a) {
        int b = sqrt(n - a * a);
        if (a * a + b * b == n) {
            return !!a + !!b;
        }
    }
    return 3;
}
```
runtime: 100.00%  
memory usage: 100.00%  

## 解法 4
和解法1，2一样，只是生成动态数组里面只有一个数字，后面推入。速度加快，因为数组内元素更少。
```c++
int numSquares(int n)
{
	vector<int> dp(1, 0);
	while (dp.size() <= n){
		int m = dp.size(), val = INT_MAX;
		for (int i = 1; i * i <= m; ++i)
			val = min(val, dp[m - i * i] + 1);
		dp.push_back(val);
	}
	return dp.back();
}
```
runtime: 78.86%  
memory usage: 11.54%  

# Link
[leetcode-279](https://leetcode.com/problems/perfect-squares/)  
[solution 3&4](https://www.cnblogs.com/grandyang/p/4800552.html)  
[四平方和定理](https://zh.wikipedia.org/wiki/%E5%9B%9B%E5%B9%B3%E6%96%B9%E5%92%8C%E5%AE%9A%E7%90%86)