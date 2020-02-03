---
layout: post
title: "Leetcode576-Out of Boundary Paths"
subtitle: 'Leetcode-recursion,dynamic programming,BFS'
date:       2019-11-25 09:36:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - recursion
  - dynamic programming
  - BFS
---
## 问题描述
There is an **m** by **n** grid with a ball. Given the start coordinate **(i,j)** of the ball, you can move the ball to **adjacent** cell or cross the grid boundary in four directions (up, down, left, right). However, you can **at most** move **N** times. Find out the number of paths to move the ball out of grid boundary. The answer may be very large, return it after mod 109 + 7.
### Example 1:
![example1](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-25/leetcode576-example1.PNG)
```c++
Input: m = 2, n = 2, N = 2, i = 0, j = 0
Output: 6
```
### Example 2:
![example2](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-25/leetcode576-example2.PNG)
```c++
Input: m = 1, n = 3, N = 3, i = 0, j = 1
Output: 12
```
### Note:
- Once you move the ball out of boundary, you cannot move it back.
- The length and height of the grid is in range [1,50].
- N is in range [0,50].

## 解法
### 解法 1
&emsp;这题和[weekly contest 164 第四题](https://hexinlin.top/2019/11/24/leetcode-Weekly-Contest-164/)神似。给出一个mxn的网格，有一个球初始
处于(i, j)位置，球每次移动时只能沿上下左右某个方向移动一个网格，求出在N步内移到边界外的方法数。  
&emsp;先用递归来解，因为移动后只是步数和当前位置变了，子问题性质和原问题性质还是一样的，适合用递归解。如果移出边界外就返回1，走完N步还没到边界外就返回0，否则就要加上沿上下左右走一步后所处位置的次数，为了防止数值越界，每次都要取余。  
&emsp;为了减少重复计算，建立三维记忆数组。
```c++
int helper(int m, int n, int N, int i, int j, vector<vector<vector<int>>>& map)
{
	if (N < 0)
		return 0;
	if (i < 0 || i >= m || j < 0 || j >= n)
		return 1;
	if (map[N][i][j] != - 1)
		return map[N][i][j];
	int res = 0;
	res += helper(m, n, N - 1, i - 1, j, map);
	res %= (int)(1e9 + 7);
	res += helper(m, n, N - 1, i + 1, j, map);
	res %= (int)(1e9 + 7);
	res += helper(m, n, N - 1, i, j - 1, map);
	res %= (int)(1e9 + 7);
	res += helper(m, n, N - 1, i, j + 1, map);
	res %= (int)(1e9 + 7);
	map[N][i][j] = res;
	return res;		
}

int findPaths(int m, int n, int N, int i, int j)
{
	vector<vector<vector<int>>> map(N + 1, vector<vector<int>> (m, vector<int>(n, -1)));
	return helper(m, n, N, i, j, map);
}
```
runtime: 90.61%   
memory usage: 50.00%  

### 解法 2
&emsp;能用递归肯定也能动态规划了，这里的动态数组肯定也是三维了，dp[i][r][c]表示从(r, c)位置最多走i步能走到边界外的走法数，动态数组初始化为全0。  
这里应注意在边界行列式要取1，否则会导致重复计算。
```c++
int findPaths(int m, int n, int N, int i, int j)
{
	vector<vector<vector<int>>> dp(N + 1, vector<vector<int>> (m, vector<int>(n, 0)));
	for (int i = 1; i <= N; ++i)	
	{
		for (int r = 0; r < m; ++r)
		{
			for (int c = 0; c < n; ++c)
			{
				long long v1 = (r == 0) ? 1 : dp[i - 1][r - 1][c];													
				long long v2 = (r == m - 1) ? 1 : dp[i - 1][r + 1][c];
				long long v3 = (c == 0) ? 1 : dp[i - 1][r][c - 1];
				long long v4 = (c == n - 1) ? 1 : dp[i - 1][r][c + 1];
				dp[i][r][c] = (v1 + v2 + v3 + v4) % ((int)(1e9 + 7));
			}
		}
	}
	return dp[N][i][j];
}
```
time complexity: O(m * n * N)  
space complexity: O(m * n * N)  
runtime: 34.00%  
memory usage: 100.00%  

### 解法 3
&emsp;解法2空间优化
```c++
int findPaths4(int m, int n, int N, int i, int j)
{
	vector<vector<int>> dp(m, vector<int>(n, 0));
	for (int i = 1; i <= N; ++i)	
	{
		vector<vector<int>> t(m, vector<int>(n, 0));
		for (int r = 0; r < m; ++r)
		{
			for (int c = 0; c < n; ++c)
			{
				long long v1 = (r == 0) ? 1 : dp[r - 1][c];													
				long long v2 = (r == m - 1) ? 1 : dp[r + 1][c];
				long long v3 = (c == 0) ? 1 : dp[r][c - 1];
				long long v4 = (c == n - 1) ? 1 : dp[r][c + 1];
				t[r][c] = (v1 + v2 + v3 + v4) % ((int)(1e9 + 7));
			}
		}
		dp = t;
	}
	return dp[i][j];
}
```
time complexity: O(m * n * N)  
space complexity: O(m * n)  
runtime: 66.43%  
memory usage: 100.00%  

### 解法 4
&emsp;广度优先搜索，这里建立二维动态数组，dp[r][c]表示从初始位置走特定步数位于(r, c)位置的走法数。总共遍历N步，在每次遍历时，建立临时数组t，对dp中每个位置，都向4个方向走一步，对应位置记为(x, y),如果(x, y)位于边界外，表示此时已经到了边界外，res应该加上dp[r][c]，因为是从(r, c)位置来的。如果不到边界外，那么t[x][y]应该加上dp[r][c]，表示走到(x, y)位置的走法多了dp[r][c]种。每次遍历完把dp赋值为t。最终返回res即可。
```c++
int findPaths(int m, int n, int N, int i, int j)
{
	int res = 0;
	vector<vector<int>> dp(m, vector<int>(n, 0));
	dp[i][j] = 1;
	vector<vector<int>> dirs;
	dirs.push_back({0, -1});
	dirs.push_back({-1, 0});
	dirs.push_back({0, 1});
	dirs.push_back({1, 0});
	for (int k = 0; k < N; ++k)
	{
		vector<vector<int>> t(m, vector<int>(n, 0));
		for (int r = 0; r < m; ++r)
		{
			for (int c = 0; c < n; ++c)
			{
				for (vector<int>& dir : dirs)
				{
					int x = r + dir[0], y = c + dir[1];
					if (x < 0 || x >= m || y < 0 || y >= n){
						res = (res + dp[r][c]) % 1000000007;
					}
					else 
						t[x][y] = (t[x][y] + dp[r][c]) % 1000000007;
				}
			}
		}
		dp = t;
	}
	return res;
}
```
time complexity: O(m * n * N)  
space complexity: O(m * n )  
runtime: 25.89%  
memory usage: 50.00%  

## Link
[leetcode-576](https://leetcode.com/problems/out-of-boundary-paths/)  
[solution 2, 3](https://www.cnblogs.com/grandyang/p/6927921.html)  