---
layout: post
title: "Leetcode650-2 Keys Keyboard"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-13 15:42:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - recursion
---

# 问题描述
Initially on a notepad only one character 'A' is present. You can perform two operations on this notepad for each step:
1. ``Copy All:`` You can copy all the characters present on the notepad (partial copy is not allowed).
2. ``Paste:`` You can paste the characters which are copied **last time**.

Given a number ``n``. You have to get **exactly** n '**A**' on the notepad by performing the minimum number of steps permitted. Output the minimum number of steps to get ``n`` '**A**'.
### Example 1:
```c++
Input: 3
Output: 3
Explanation:
Intitally, we have one character 'A'.
In step 1, we use Copy All operation.
In step 2, we use Paste operation to get 'AA'.
In step 3, we use Paste operation to get 'AAA'.
```
### Note:
1. The ``n`` will be in the range [1, 1000].

# 解法
## 解法 1
这种题目一看就是动态规划啦。动态数组``dp[i]``表示i个A需要最少的操作次数。接下来就是要求状态转移方程，因为每次只能复制全部，那么当A的个数为可除时才能被粘贴得到，至于具体从那个个数粘贴几次就是动态规划的问题了。状态转移方程为当i能够被j整除时， ``dp[i] = min(dp[i], (j - 1) + 1 + dp[i / j])``，其中因为当前是j倍，只需粘贴j-1次，粘贴之前先要复制一次所以要(j - 1) + 1，还要加上复制前达到i/j需要的次数。
```c++
int minSteps(int n)
{
	vector<int> dp(n + 1, n + 1);
	dp[0] = 0;
	dp[1] = 0;
	for (int i = 2; i <= n; ++i)
	{	
		for (int j = 2; j <= i; ++j)
		{
			if (i % j == 0){
				dp[i] = min(dp[i], (j - 1) + 1 + dp[i / j]);
			}
		}
		
	}
	return dp.back();
}
```
time complexity: O(n^2)  
space complexity: O(n)  
runtime: 14.67%  
memory usage: 25.00%  


## 解法 2
用动态规划均能递归，这里递归竟然还更快。
```c++
int minSteps(int n)
{
	if (n == 1)
		return 0;
	int res = n;
	for (int i = n - 1; i >= 2; --i)
	{
		if (n % i == 0)
			res = min(res, minSteps4(n / i) + i);
	}
	return res;
}
```
runtime: 76.03%  
memory usage: 87.50%  

## 解法 3
之前动态规划时，倍数是从2递增到i被，这里从最大倍数递减，当遇到整除时，就break，因为当前这个就是最小，证明我也不会啊，可能和
尽量多粘贴有关。i个A最多需要i步操作达到。
```c++
int minSteps(int n)
{
	vector<int> dp(n + 1, 0);
	for (int i = 2; i <= n; ++i)
	{
		dp[i] = i;
		for (int j = n - 1; j >= 2; --j)
		{
			if (i % j == 0){
				dp[i] = min(dp[i], dp[i / j] + j);
				break;
			}
		}
	}
	return dp.back();
}
```

```c++
int minSteps(int n)
{
	vector<int> dp(n + 1, 0);
	for (int i = 2; i <= n; ++i)
	{
		dp[i] = i;
		for (int j = 2; j < n; ++j)
		{
			if (i % j == 0){
				dp[i] = min(dp[i], dp[j] +  i / j);
				break;
			}
		}
	}
	return dp.back();
}
```
这两种写法都是一样的意思。   
time complexity: O(n^2)  
space complexity: O(n)  
runtime: 36.30%  
memory usage: 25.00%  

```c++
    int minSteps(int n)
    {
        vector<int> dp(n + 1, 0);
        for (int i = 2; i <= n; ++i)
        {
            dp[i] = i;
            for (int j = n - 1; j > 1; --j)
            {
                if (i % j == 0){
                    dp[i] = min(dp[i], dp[j] +  i / j);
                    break;
                }
            }
        }
        return dp.back();
    }
```
这种写法表示多的粘贴少数次，是不行的！！！

## 解法 4
空间优化
```c++
int minSteps(int n)
{
	int res = 0;
	for (int i = 2; i <= n; ++i)
	{
		while (n % i == 0){
			res += i;
			n /= i;
		}
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 100.00%  
memory usage: 75.00%  

# Link
[leetcode-650](https://leetcode.com/problems/2-keys-keyboard/)  
[solution 2-4](https://www.cnblogs.com/grandyang/p/7439616.html)  