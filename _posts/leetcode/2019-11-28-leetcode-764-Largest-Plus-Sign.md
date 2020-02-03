---
layout: post
title: "Leetcode764-Largest Plus Sign"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-28 11:13:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---
# 问题描述
In a 2D ``grid`` from (0, 0) to (N-1, N-1), every cell contains a ``1``, except those cells in the given list ``mines`` which are ``0``. What is the largest axis-aligned plus sign of ``1``s contained in the grid? Return the order of the plus sign. If there is none, return 0.

An "``axis-aligned plus sign of 1s of order **k**``" has some center ``grid[x][y] = 1`` along with 4 arms of length k-1 going up, down, left, and right, and made of ``1`` s. This is demonstrated in the diagrams below. Note that there could be ``0`` s or ``1`` s beyond the arms of the plus sign, only the relevant area of the plus sign is checked for ``1`` s.
### Examples of Axis-Aligned Plus Signs of Order k:
```c++
Order 1:
000
010
000

Order 2:
00000
00100
01110
00100
00000

Order 3:
0000000
0001000
0001000
0111110
0001000
0001000
0000000
```
### Example 1:
```c++
Input: N = 5, mines = [[4, 2]]
Output: 2
Explanation:
11111
11111
11111
11111
11011
In the above grid, the largest plus sign can only be order 2.  One of them is marked in bold.
```
### Example 2:
```c++
Input: N = 2, mines = []
Output: 1
Explanation:
There is no plus sign of order 2, but there is of order 1.
```
### Example 3:
```c++
Input: N = 1, mines = [[0, 0]]
Output: 0
Explanation:
There is no plus sign, so return 0.
```
### Note:
1. ``N`` will be an integer in the range [``1, 500``].
2. ``mines`` will have length at most ``5000``.
3. ``mines[i]`` will be length 2 and consist of integers in the range [``0, N-1``].
4. (Additionally, programs submitted in C, C++, or C# will be judged with a slightly smaller time limit.)

# 解法
## 解法 1
&emsp;这道题就是要寻找最大的由1组成的"+"号，给出一个大小为N的网格，mines数组给出了其中的"0"的位置。  
&emsp;加号肯定是有上下左右四个方向组成，其中长度最短的方向限制加号的大小。用动态数组left,right,up,down分别表示左右上下四个放下连续1的长度，每个位置加号就是其中最小值。但是遍历grid过程中一次最多只能遍历两个方向，左和上，所以需要两次遍历，一次从左上角到右下角，更新left和up，另一次从右下角到左上角，更新right和down。
```c++
int orderOfLargestPlusSign(int N, vector<vector<int>>& mines)
{
	int res = 0;
	vector<vector<int>> grid(N, vector<int>(N, 1));
	for (auto& ele : mines)
		grid[ele[0]][ele[1]] = 0;
	vector<vector<int>> left(N, vector<int>(N, 0)), right(N, vector<int>(N, 0)), up(N, vector<int>(N, 0)), down(N, vector<int>(N, 0));
	for (int i = 0; i < N; ++i)
	{
		for (int j = 0; j < N; ++j)
		{
			if (grid[i][j] == 0){
				left[i][j] = 0;
				up[i][j] = 0;
			}
			else {
				left[i][j] = 1;
				if (j > 0)
					left[i][j] += left[i][j - 1];
				up[i][j] = 1;
				if (i > 0)
					up[i][j] += up[i - 1][j];
			}
		}
	}
	for (int i = N - 1; i >= 0; --i)
	{
		for (int j = N - 1; j >= 0; --j)
		{
			if (grid[i][j] == 0){
				right[i][j] = 0;
				down[i][j] = 0;
			}
			else {
				right[i][j] = 1;
				if (j < N - 1)
					right[i][j] += right[i][j + 1];
				down[i][j] = 1;
				if (i < N - 1)
					down[i][j] += down[i + 1][j];
				res = max(res, min(left[i][j], min(right[i][j], min(up[i][j], down[i][j]))));
			}
		}
	}
	return res;
}
```
time compelxity: O(N * N)  
space complexity: O(N * N)  
runtime:  53.28%  
memory usage: 50.00%  

## 解法 2
&emsp;其实这题动态数组只用一个即可，反正是最小的形成限制，按最小来更新即可。
```c++
int orderOfLargestPlusSign2(int N, vector<vector<int>>& mines)
{
	int res = 0;
	vector<vector<int>> grid(N, vector<int>(N, 1));
	for (auto& ele : mines)
		grid[ele[0]][ele[1]] = 0;
	vector<vector<int>> dp(N, vector<int>(N, 0));
	for (int j = 0; j < N; ++j)
	{
		int cnt = 0;
		for (int i = 0; i < N; ++i)
		{
			cnt = grid[i][j] ? cnt + 1 : 0;
			dp[i][j] = cnt;
		}
		cnt = 0;
		for (int i = N - 1; i >= 0; --i)
		{
			cnt = grid[i][j] ? cnt + 1 : 0;
			dp[i][j] = min(dp[i][j], cnt);
		}
	}
	for (int i = 0; i < N; ++i)
	{
		int cnt = 0;
		for (int j = 0; j < N; ++j)
		{
			cnt = grid[i][j] ? cnt + 1 : 0;
			dp[i][j] = min(dp[i][j], cnt);
		}
		cnt = 0;
		for (int j = N - 1; j >= 0; --j)
		{
			cnt = grid[i][j] ? cnt + 1 : 0;
			dp[i][j] = min(dp[i][j], cnt);
			res = max(res, dp[i][j]);
		}
	}
	return res;
}
```
time compelxity: O(N * N)  
space complexity: O(N * N)  
runtime:  73.54%  
memory usage: 50.00%  

## 解法 3
&emsp;这个解法更加精简，用四个变量l,r,u,d存储连续长度，遍历一次即可更新完所有方向。
```c++
int orderOfLargestPlusSign3(int N, vector<vector<int>>& mines)
{
	int res = 0;
	vector<vector<int>> dp(N, vector<int>(N, N));
	for (auto ele : mines)
		dp[ele[0]][ele[1]] = 0;
	for (int i = 0; i < N; ++i)
	{
		int l = 0, r = 0, u = 0, d = 0;
		for (int j = 0, k = N - 1; j < N; ++j, --k)
		{
			dp[i][j] = min(dp[i][j], l = dp[i][j] ? l + 1 : 0);
			dp[i][k] = min(dp[i][k], r = dp[i][k] ? r + 1 : 0);
			dp[j][i] = min(dp[j][i], u = dp[j][i] ? u + 1 : 0);
			dp[k][i] = min(dp[k][i], d = dp[k][i] ? d + 1 : 0);
		}
	}
	for (int k = 0; k < N * N; ++k)
		res = max(res, dp[k / N][k % N]);
	return res;
}
```
time compelxity: O(N * N)  
space complexity: O(N * N)  
runtime:  75.91%  
memory usage: 100.00%  

# Link
[leetcode-764](https://leetcode.com/problems/largest-plus-sign/)  
[solution 2, 3](https://www.cnblogs.com/grandyang/p/8679286.html)  

