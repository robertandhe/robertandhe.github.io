---
layout: post
title: "Leetcode688-Kinight Probability in Chessboard"
subtitle: 'Leetcode-dynamic programming, BFS'
date:       2019-11-22 20:44:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
  - BFS
---

# 问题描述
On an ``NxN`` chessboard, a knight starts at the ``r``-th row and ``c``-th column and attempts to make exactly ``K ``moves. The rows and columns are 0 indexed, so the top-left square is ``(0, 0)``, and the bottom-right square is ``(N-1, N-1)``.

A chess knight has 8 possible moves it can make, as illustrated below. Each move is two squares in a cardinal direction, then one square in an orthogonal direction.

Each time the knight is to move, it chooses one of eight possible moves uniformly at random (even if the piece would go off the chessboard) and moves there.

The knight continues moving until it has made exactly K moves or has moved off the chessboard. Return the probability that the knight remains on the board after it has stopped moving.

### Example:
```c++
Input: 3, 2, 0, 0
Output: 0.0625
Explanation: There are two moves (to (1,2), (2,1)) that will keep the knight on the board.
From each of those positions, there are also two moves that will keep the knight on the board.
The total probability the knight stays on the board is 0.0625.
```
### Note:
- ``N`` will be between 1 and 25.
- ``K`` will be between 0 and 100.
- The knight always initially starts on the board.

# 解法
## 解法 1
这道题给出一个NxN大小的棋盘，一个骑士初始位于(r, c)位置，骑士只能朝8个方向走‘日’字，问走K步后骑士还位于棋盘上的概率。
这个概率就是走K步还位于棋盘上的数量除以总数。总数就是8的K次方了，加下来就是求走K步后还位于棋盘上的总情形数，这里采用深度优先搜索，对每次递归，当前位置(i, j)的总情形数不就是往8个方向走到的总情形数的和。最后返回初始位置总情形数除以总数。  
这里还有优化问题要考虑，如果仅仅无脑递归，很容易TLE，因为有许多走法到某个相同位置时正好还剩同样的步数，这就是重复计算，所以需要记忆数组来减少重复计算，这里既有剩余步数，还是棋盘上的位置，所以记忆数组是3维，初始化为0。

```c++
double helper(int N, int K, int r, int c, vector<vector<vector<double>>>& memo)
{
	if (r < 0 || r >= N || c < 0 || c >= N)
		return 0;
	if (K == 0)
		return 1.0;
	if (memo[K][r][c] != 0)
		return memo[K][r][c];
	vector<vector<int>> offset;   // 为了防止jekyll无法识别，只能这种写法
	offset.push_back({-1, -2});
	offset.push_back({-2, -1});
	offset.push_back({-2, 1});	
	offset.push_back({-1, 2});
	offset.push_back({1, 2});
	offset.push_back({2, 1});
	offset.push_back({2, -1});
	offset.push_back({1, -2});
	double res = 0;
	for (int i = 0; i < offset.size(); ++i)
	{
		res += helper(N, K - 1, r + offset[i][0], c + offset[i][1], memo);
	}
	memo[K][r][c] = res;
	return res;
}
double knightProbability(int N, int K, int r, int c)
{
	vector<vector<vector<double>>> memo(K + 1, vector<vector<double>> (N, vector<double>(N)));
	double res = helper(N, K, r, c, memo);
	return res / pow(8, K);
}
```
runtime: 5.84%  
memory usage: 100.00%  

## 解法 2
既然能用带记忆数组的递归就能用动态规划。可以用如解法1一样用三维动态数组也可以用二维，每次更新即可，这里用二维。动态数组定义为dp[i][j]表示(i, j)位置走相应步数还在棋盘上的情形数，初始化肯定是全为1了。那么如何更新，因为需要统计往8个方向走后还在棋盘上的总情形数，初始化时肯定是全为0了，走后加上走到的位置数量即可。
```c++
double knightProbability(int N, int K, int r, int c)
{
	vector<vector<int>> offset;  // 为了防止jekyll无法识别，只能这种写法
	offset.push_back({-1, -2});
	offset.push_back({-2, -1});
	offset.push_back({-2, 1});	
	offset.push_back({-1, 2});
	offset.push_back({1, 2});
	offset.push_back({2, 1});
	offset.push_back({2, -1});
	offset.push_back({1, -2});
	vector<vector<double>> dp(N, vector<double>(N, 1));
	for (int m = 0; m < K; ++m)
	{
		vector<vector<double>> t(N, vector<double>(N, 0));
		for (int i = 0; i < N; ++i)
		{
			for (int j = 0; j < N; ++j)
			{
				for (auto& dir : offset)
				{
					int x = i + dir[0], y = j + dir[1];
					if (x < 0 || x >= N || y < 0 || y >= N)
						continue;
					t[i][j] += dp[x][y];
				}
			}
		}
		dp = t;
	}
	return dp[r][c] / pow(8, K);
}
```
time complexity: O(N^2 * K)  
space complexity: O(N * N)  
runtime: 54.79%  
memory usage: 100.00%   


# Link
[leetcode-688](https://leetcode.com/problems/knight-probability-in-chessboard/)  
[solution 2](https://www.cnblogs.com/grandyang/p/7639153.html)