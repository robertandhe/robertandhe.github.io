---
layout: post
title: "Leetcode1039 - Minimum Score Triangulation of Polygon"
subtitle: 'Leetcode1039-dynamic programming'
date:       2019-12-04 20:05:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
Given ``N``, consider a convex N-sided polygon with vertices labelled ``A[0], A[i], ..., A[N-1]`` in clockwise order.

Suppose you triangulate the polygon into ``N-2`` triangles.  For each triangle, the value of that triangle is the **product** of the labels of the vertices, and the total score of the triangulation is the sum of these values over all ``N-2`` triangles in the triangulation.

Return the smallest possible total score that you can achieve with some triangulation of the polygon.

### Example 1:
```c++
Input: [1,2,3]
Output: 6
Explanation: The polygon is already triangulated, and the score of the only triangle is 6.
```
### Example 2:
![leetcode-1039](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-12-04/leetcode1039.PNG)
```c++
Input: [3,7,4,5]
Output: 144
Explanation: There are two triangulations, with possible scores: 3*7*5 + 4*5*7 = 245, or 3*4*5 + 3*4*7 = 144.  The minimum score is 144.
```
### Example 3:
```c++
Input: [1,3,1,4,1,5]
Output: 13
Explanation: The minimum score triangulation has score 1*1*3 + 1*1*4 + 1*1*5 + 1*1*1 = 13.
```
### Note:
1. ``3 <= A.length <= 50``
2. ``1 <= A[i] <= 100``

# 解法
&emsp;这道题给出一多边形顶点上的数值，在多边形中划分三角形，问划分出的三角形顶点积的和的最小值。 
&emsp;题目中也给出了提示，对N个顶点划分三角形时，A[0]和A[N - 1]肯定是共同在某一些三角形内，至于另一个顶点是哪个就需要遍历[1, N - 2]中所有顶点。设另一个顶点为第k个，那么A[0], A[N - 1], A[k]组成一个三角形，另外两块区分就是动态规划子问题。  
&emsp;这里设动态数组dp[i][j]表示第i个到第j个顶点划分出的最小值。dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j] + A[i] * A[j] * A[k]),其中k遍历范围为[i + 1, j - 1]。  
&emsp;只有三个以上顶点才构成多边形，i,j的差值从2逐渐增大到n - 2，遍历所有情况。返回值为dp[0][n-1]。
```c++
int minScoreTriangulation(vector<int>& A) 
{
	int n = A.size();
	vector<vector<int>> dp(n, vector<int>(n));
	for (int d = 2; d < n; ++d)
	{
		for (int i = 0; i + d < n; ++i)
		{
			int j = i + d;
			int mn = INT_MAX;
			for (int k = i + 1; k < j; ++k)
				mn = min(mn, dp[i][k] + dp[k][j] + A[i] * A[j] * A[k]);
			if (mn != INT_MAX)
				dp[i][j] = mn;
		}
	}
	return dp[0][n - 1];
}
```
time complexity: O(n^3)  
space complexity: O(n^2)  
runtime: 97.44%  
memory usage: 100.00%  

# Link
[leetcode-1039](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/)  
[solution](http://www.noteanddata.com/leetcode-1039-Minimum-Score-Triangulation-of-Polygon-java-solution-note.html)