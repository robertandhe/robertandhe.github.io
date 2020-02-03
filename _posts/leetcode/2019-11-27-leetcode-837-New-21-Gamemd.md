---
layout: post
title: "Leetcode 837 - New 21 Game"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-27 18:10:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Alice plays the following game, loosely based on the card game "21".

Alice starts with ``0`` points, and draws numbers while she has less than ``K`` points.  During each draw, she gains an integer number of points randomly from the range [``1, W``], where ``W`` is an integer.  Each draw is independent and the outcomes have equal probabilities.

Alice stops drawing numbers when she gets ``K`` or more points.  What is the probability that she has N or less points?
### Example 1:
```c++
Input: N = 10, K = 1, W = 10
Output: 1.00000
Explanation:  Alice gets a single card, then stops.
```
### Example 2:
```c++
Input: N = 6, K = 1, W = 10
Output: 0.60000
Explanation:  Alice gets a single card, then stops.
In 6 out of W = 10 possibilities, she is at or below N = 6 points.
```
### Example 3:
```c++
Input: N = 21, K = 17, W = 10
Output: 0.73278
```
### Note:
1. ``0 <= K <= N <= 10000``
2. ``1 <= W <= 10000``
3. Answers will be accepted as correct if they are within ``10^-5`` of the correct answer.
4. The judging time limit has been reduced for this question.

# 解法
## 解法 1
题目就是21点游戏，每次等概率选出[1, W]范围内整数，大于等于K是不再抽，问最终小于等于N的概率。  
这道题我首先想的是求出停止时取各个数的概率，再求出大于等于K并且小于等于N的概率。动态方程长度肯定是K+W了，因为大于等于K时就不抽了，每次最多抽到W，所以能得到的最大值为K+W-1。dp[i]就表示取到i的概率了。  
那么状态转移方程如何求，这里就是全概率公式了，i可以由之前取到i-1,再抽一个1得到，也可以由之前取到i-2,再抽一个2得到...，也可以由之前取到i-W,再取一个W得到。最多往后退W个，因为可以抽的最大数为W。在这里注意下大于等于K是不能抽，假设之前取j，想得到i需要再抽一个i-j,这个数的概率等于1/W，所以dp[i] += dp[j] / W。  
这题代码是没问题的，可惜TLE了。
```c++
double new21Game(int N, int K, int W)
{
	vector<double> dp(K + W, 0);
	dp[0] = 1.0;
	for (int i = 0; i < K + W; ++i)
	{
		for (int j = i - 1; j >= 0 && j >= i - W ; --j)
		{
			if (j < K)
				dp[i] += dp[j] / W;
		}		
	}
	double n = 0, d = 0;
	for (int i = K; i < K + W; ++i)
	{
		d += dp[i];
		if (i <= N)
			n += dp[i];
	}
	return n / d;
}
```

## 解法 2
这道题换了个思路，动态数组sum[i]表示取到[0, i]范围内数的概率之和。即sum[i] = p[0] + p[1] + p[2] + ... + p[i]。  
状态转移方程需要分情况来讨论：
- 如果i <= W，那么j可以从i-1推到0，因为抽的最大数不会超过W。例如 p[7] = 1 / W * (p[6] + p[5] + ... + p[0])。7可以由6加1，5加2，... ，0加7得到。所以p[i] = 1/ W * sum[i - 1], sum[i] = sum[i - 1] + p[i] = sum[i - 1] + sum[i - 1] / W；
- 如果i > W, 那么j最多可以从i - 1推到i - W。例如W = 10, p[15] = 1 / W * (p[14] + p[13] + p[12] + ... + p[5])。即p[i] = 1/ W * (sum[i - 1] - sum[i - W - 1])。sum[i] = sum[i - 1] + p[i] = sum[i - 1] + (sum[i - 1] - sum[i - W - 1]) / W;

但是这里还有一个限制条件，j大于等于K是就不能再抽了，所以上述状态转移方程应该改为：
- 如果i <= W并且i <= K，那么j可以从i-1推到0，因为抽的最大数不会超过W。例如 p[7] = 1 / W * (p[6] + p[5] + ... + p[0])。7可以由6加1，5加2，... ，0加7得到。所以p[i] = 1/ W * sum[i - 1], sum[i] = sum[i - 1] + p[i] = sum[i - 1] + sum[i - 1] / W；
- 如果i > W并且i <= K, 那么j最多可以从i - 1推到i - W。例如W = 10, p[15] = 1 / W * (p[14] + p[13] + p[12] + ... + p[5])。即p[i] = 1/ W * (sum[i - 1] - sum[i - W - 1])。sum[i] = sum[i - 1] + p[i] = sum[i - 1] + (sum[i - 1] - sum[i - W - 1]) / W;

当i > K时，j不能从i - 1开始，j至多能从K-1开始，所以状态转移方程还包括：
- 如果i <= W 并且i > K, p[i] = 1 / W * sum[K - 1], sum[i] = sum[i - 1] + p[i] = sum[i - 1] + sum[K - 1] / W；
- 如果i > W并且 i > K, p[i] = 1 / W * (sum[K - 1] - sum[i - W - 1]), sum[i] = sum[i - 1] + p[i] = sum[i - 1] + (sum[K - 1] - sum[i - W - 1]) / W。

在下面的代码中，把K和W合二为一，因为我们也不知道K和W谁大谁小。只需按照小的来即可。  
返回最终数大于等于K并且小于等于N的概率，由条件概率公式就是大于等于K并且小于等于N的总概率除以大于等于K的概率，所以返回(sum[N] - sum[K - 1]) / (sum[K + W - 1] - sum[K - 1])。（注：其实sum[K + W - 1] - sum[K - 1] = 1，因为最终肯定大于等于1。）
```c++
double new21Game(int N, int K, int W)
{
	if (K <= 0 || N >= K + W)
		return 1.0;
	vector<double> sum(K + W);
	sum[0] = 1.0;
	for (int i = 1; i < K + W; ++i)
	{
		int t = min(K - 1, i - 1);
		if (i <= W)
			sum[i] = sum[i - 1] + sum[t]/ W;
		else {
			sum[i] = sum[i - 1] + (sum[t] - sum[i - W - 1])  / W;
		}
	}
	return (sum[N] - sum[K - 1]) / (sum[K + W - 1] - sum[K - 1]);
}
```
time compelxity: O(K + W)  
space complexity: O(K + W)  
runtime: 35.31%  
memory usage: 90.91%   

## 解法 3
和解法2思想一模一样，只是先不区分i和K大小,加上再说，如果i > K,表示加多了，把多的减去即可。
```c++
double new21Game(int N, int K, int W)
{
	if (K <= 0 || N >= K + W)
		return 1.0;
	vector<double> dp(K + W);
	dp[0] = 1.0;
	for (int i = 1; i < K + W; ++i)
	{
		dp[i] = dp[i - 1];
		if (i <= W)
			dp[i] += dp[i - 1] / W;
		else 
			dp[i] += (dp[i - 1] - dp[i - W - 1]) / W;
		if (i > K)
			dp[i] -= (dp[i - 1] - dp[K - 1]) / W;
	}
	return dp[N] - dp[K - 1];
}
```
time compelxity: O(K + W)  
space complexity: O(K + W)  
runtime: 82.91%  
memory usage: 90.91%    

## 解法 4
和解法2，3思想大同小异，这里动态数组大小为N+1,因为我们只关心大于等于K并且小于等于N的概率。动态数组dp[i]表示取到i的概率而不是解法2，3中的累加概率。在遍历过程中如果大于等于K，表示到末尾，加入res。小于K还能继续抽。大于等于W要减去多加的。
```c++
double new21Game(int N, int K, int W)
{
	if (K <= 0 || N >= K + W)
		return 1.0;
	vector<double> dp(N + 1);
	dp[0] = 1.0;
	double res = 0, sumW = 1;
	for (int i = 1; i <= N; ++i)
	{
		dp[i] = sumW / W;
		if (i < K)
			sumW += dp[i];
		else 
			res += dp[i];
		if (i >= W)
			sumW -= dp[i - W];
	}
	return res;
}
```
time compelxity: O(N)  
space complexity: O(N)  
runtime:  35.31%  
memory usage: 100.00%   

# Link
[leetcode-837](https://leetcode.com/problems/new-21-game/)  
[solution 2-4](https://www.cnblogs.com/grandyang/p/10386525.html)  