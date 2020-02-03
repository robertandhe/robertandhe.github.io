---
layout: post
title: "Leetcode Weekly Contest 164"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-24 11:41:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - Math
  - string
  - recursion
---


>写在前面，这是第二次参加leetcode weekly contest，这次终于把题目做完了，全球排名548/5820，还有提升空间。

# 第一题：5271. Minimum Time Visiting All Points(Easy)
## 题目描述
On a plane there are ``n`` points with integer coordinates ``points[i] = [xi, yi]``. Your task is to find the minimum time in seconds to visit all points.
You can move according to the next rules:
- In one second always you can either move vertically, horizontally by one unit or diagonally (it means to move one unit vertically and one unit horizontally in one second).
- You have to visit the points in the same order as they appear in the array.

#### Example 1:
![leetcode5271-example1](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-24/leetcode5271.PNG){:height="50%" width="50%"}
```c++
Input: points = [[1,1],[3,4],[-1,0]]
Output: 7
Explanation: One optimal path is [1,1] -> [2,2] -> [3,3] -> [3,4] -> [2,3] -> [1,2] -> [0,1] -> [-1,0]   
Time from [1,1] to [3,4] = 3 seconds 
Time from [3,4] to [-1,0] = 4 seconds
Total time = 7 seconds
```
#### Example 2:
```c++
Input: points = [[3,2],[-2,2]]
Output: 5
```
#### Constraints:
- ``points.length == n``
- ``1 <= n <= 100``
- ``points[i].length == 2``
- ``-1000 <= points[i][0], points[i][1] <= 1000``

## 解法
&emsp;这题给出一系列点，要求出把它们连起来需要的最小移动步数，可以向上下左右移动一个单位或者沿对角线移动一个单位。
&emsp;那么我们肯定是优先沿对角线移动，因为沿对角线移动在同时减小x方向和y方向的差距，当没法沿对角线移动时再考虑沿着
垂直或者水平方向移动，至于到底是沿垂直还是水平就要看那个方向差距大而无法通过对角线移动弥补。  
&emsp;这题限定要沿着给出的点一个个连起来，如果没有这个条件难度陡然上升。
```c++
int minTimeToVisitAllPoints(vector<vector<int>>& points) 
{
   int res = 0, n = points.size();     
   for (int i = 0; i < n - 1; ++i)
   {
	   int deltaX = abs(points[i + 1][0] - points[i][0]);
	   int deltaY = abs(points[i + 1][1] - points[i][1]);
	   int delta = min(deltaX, deltaY);
	   res += delta;
	   if (deltaX < deltaY){
		   res += deltaY - delta;
	   }
	   else 
	   	res += deltaX - delta;
   }
   return res;
}
```


# 第二题：5272. Count Servers that Communicate(Medium)
## 题目描述：
You are given a map of a server center, represented as a ``m * n`` integer matrix grid, where 1 means that on that cell there is a server and 0 means that it is no server. Two servers are said to communicate if they are on the same row or on the same column.

Return the number of servers that communicate with any other server.
#### Example 1：
![leetcode5272-example1](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-24/leetcode5272-1.PNG)
```c++
Input: grid = [[1,0],[0,1]]
Output: 0
Explanation: No servers can communicate with others.
```
#### Example 2：
![leetcode5272-example2](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-24/leetcode5272-2.PNG)
```c++
Input: grid = [[1,0],[1,1]]
Output: 3
Explanation: All three servers can communicate with at least one other server.
```
#### Example 3：
![leetcode5272-example3](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-24/leetcode5272-3.PNG){:height="70%" width="70%"}
```c++
Input: grid = [[1,1,0,0],[0,0,1,0],[0,0,1,0],[0,0,0,1]]
Output: 4
Explanation: The two servers in the first row can communicate with each other. The two servers in the third column can communicate with each other. The server at right bottom corner can't communicate with any other server.
```

#### Constraints:
- ``m == grid.length``
- ``n == grid[i].length``
- ``1 <= m <= 250``
- ``1 <= n <= 250``
- ``grid[i][j] == 0 or 1``

## 解法
&emsp;这题给出一个网络grid，网络中每个节点为1是代表那个位置有服务器，为0则没有，同一行或者同一列中有服务器时表示这个服务器和别的服务器相连，要求出网络中和别的服务器相连的服务器数量。这里直接遍历，对相应位置为1时则可以统计那行那列有没有服务器，为0则继续。
```c++
int countServers(vector<vector<int>>& grid)
{
	int res = 0, m = grid.size(), n = grid[0].size();
	for (int i = 0; i < m; ++i)
	{
		for (int j = 0; j < n; ++j)
		{
			if (grid[i][j] != 1)
				continue;
			int flag = 1;
			for (int r = 0; r < m; ++r)
			{
				if (r == i)
					continue;
				if (grid[r][j] == 1){
					++res;
					flag = 0;
					break;
				}
			}
			for (int c = 0; flag && c < n; ++c)
			{
				if (c == j)
					continue;
				if (grid[i][c] == 1){
					++res;
					break;
				}
			}
		}
	}
	return res;
}
```


# 第三题：1268. Search Suggestions System
## 问题描述
Given an array of strings ``products`` and a string ``searchWord``. We want to design a system that suggests at most three product names from ``products`` after each character of ``searchWord`` is typed. Suggested products should have common prefix with the searchWord. If there are more than three products with a common prefix return the three lexicographically minimums products.

Return list of lists of the suggested ``products`` after each character of ``searchWord`` is typed. 

#### Example 1:
```c++
Input: products = ["mobile","mouse","moneypot","monitor","mousepad"], searchWord = "mouse"
Output: [
["mobile","moneypot","monitor"],
["mobile","moneypot","monitor"],
["mouse","mousepad"],
["mouse","mousepad"],
["mouse","mousepad"]
]
Explanation: products sorted lexicographically = ["mobile","moneypot","monitor","mouse","mousepad"]
After typing m and mo all products match and we show user ["mobile","moneypot","monitor"]
After typing mou, mous and mouse the system suggests ["mouse","mousepad"]
```
#### Example 2:
```c++
Input: products = ["havana"], searchWord = "havana"
Output: [["havana"],["havana"],["havana"],["havana"],["havana"],["havana"]]
```
#### Example 3:
```c++
Input: products = ["bags","baggage","banner","box","cloths"], searchWord = "bags"
Output: [["baggage","bags","banner"],["baggage","bags","banner"],["baggage","bags"],["bags"]]
```
#### Example 4:
```c++
Input: products = ["havana"], searchWord = "tatiana"
Output: [[],[],[],[],[],[],[]]
```
#### Constraints:
- ``1 <= products.length <= 1000``
- ``1 <= Σ products[i].length <= 2 * 10^4``
- All characters of ``products[i]`` are lower-case English letters.
- ``1 <= searchWord.length <= 1000``
- All characters of ``searchWord ``are lower-case English letters.

## 解法
&emsp;题目要求对输入的检索词，给出搜索推荐。每输入一个字符给出至多3个推荐检索词，要求它们的前缀和现在已经输入的检索词相同，当有多余3个词前缀和已经输入词相同时，给出字典序前3个。  
&emsp;那么首先就肯定要用sort给字符串排序了，先建立空字符串str，每输入一个字符都加入字符串str,然后遍历products，找到前缀和str相同的字符串，多余3个时取前3个，这里用cnt统计已经找到的个数。
```c++
vector<vector<string>> suggestedProducts(vector<string>& products, string searchWord) 
{
	vector<vector<string>> res;
	sort(products.begin(), products.end());
	string str = "";
	for (char& c : searchWord)
	{
		vector<string> t;
		str.push_back(c);
		int cnt = 0;
		for (int i = 0; i < products.size() && cnt < 3; ++i)
		{
			string product = products[i];
			if (product.substr(0, str.size()) == str && cnt < 3){
				t.push_back(product);
				++cnt;
			}
		}
		res.push_back(t);
	}
	return res;
}
```

# 第四题：1269. Number of Ways to Stay in the Same Place After Some Steps
## 题目描述：
You have a pointer at index ``0`` in an array of size ``arrLen``. At each step, you can move 1 position to the left, 1 position to the right in the array or stay in the same place  (The pointer should not be placed outside the array at any time).

Given two integers ``steps`` and ``arrLen``, return the number of ways such that your pointer still at index ``0`` after exactly ``steps`` steps.

Since the answer may be too large, return it modulo ``10^9 + 7``.
#### Example 1:
```c++
Input: steps = 3, arrLen = 2
Output: 4
Explanation: There are 4 differents ways to stay at index 0 after 3 steps.
Right, Left, Stay
Stay, Right, Left
Right, Stay, Left
Stay, Stay, Stay
```
#### Example 2:
```c++
Input: steps = 2, arrLen = 4
Output: 2
Explanation: There are 2 differents ways to stay at index 0 after 2 steps
Right, Left
Stay, Stay
```
#### Example 3:
```c++
Input: steps = 4, arrLen = 2
Output: 8
```
#### Constraints:
- ``1 <= steps <= 500``
- ``1 <= arrLen <= 10^6``

## 解法
&emsp;开始位于``0``位置，有长度为``arrLen``的数组，可以走``steps``步，每一步可以选择向左走单位，或者向右走一单位或者待在原地，要求出走``steps``步后最终位于0位置的走法数量。  
&emsp;这题一看就是递归了，因为每走一步只是减少了剩余的步数还有所处位置，子问题性质和原问题一模一样。假设当前还剩steps步，位于pos位置，如果选择向右走，就还剩steps-1步，当前位置到了pos+1；如果选择向左走，就还剩steps-1步，当前位置到了pos-1；如果选择待在原地，就还剩steps-1步，当前位置还是pos。  
&emsp;递归过程终止条件就是如果超出界限，即pos < 0 或者 pos >= arrLen， 则返回0；如果剩余步数steps < 0,也要返回0；如果steps=0并且pos=0,即走完了所有步数并且位于0位置，则符合条件返回1。  
&emsp;为了减少重复计算，应该使用记忆数组记忆中间状态，即到某些相同位置时剩余可能也一样，那么应该直接返回值，避免重复计算。既要存储剩余步数，又要存储当前位置，记忆数组肯定是二维啦。注意这里记忆数组应该初始化为-1，不能初始化为0，因为可能某些位置不存在能够在剩余步数走回原点的走法。  
&emsp;还有一些边界情况，arrLen很大，严重大于steps，动态数组会超过内存限制，这时直接把arrLen截断为steps+1即可，因为最多能连续往右走，steps步数内最多走到steps+1位置。
```c++
int helper(int steps, int arrLen, int pos, vector<vector<int>>& map)
{
	if (pos < 0 || pos >= arrLen || steps < 0)
		return 0;
	if (steps == 0 && pos == 0)
		return 1;
	if (map[steps][pos] != -1)
		return map[steps][pos];
	int res = 0;
	res += helper(steps - 1, arrLen, pos + 1, map);
	res %= ((int)1e9 + 7);
	res += helper(steps - 1, arrLen, pos, map);
	res %= ((int)1e9 + 7);
	res += helper(steps - 1, arrLen, pos - 1, map);
	res %= ((int)1e9 + 7);
	map[steps][pos] = res;
	return res;
}

int numWays(int steps, int arrLen)
{
	int res = 0;
	if (arrLen > steps + 1)
		arrLen = steps + 1;
	vector<vector<int>> map(steps + 1, vector<int> (arrLen, -1));
	return helper(steps, arrLen, 0, map);
}
```

# Link
[1266. Minimum Time Visiting All Points](https://leetcode.com/contest/weekly-contest-164/problems/minimum-time-visiting-all-points/)  
[1267. Count Servers that Communicate](https://leetcode.com/contest/weekly-contest-164/problems/count-servers-that-communicate/)  
[1268. Search Suggestions System](https://leetcode.com/contest/weekly-contest-164/problems/search-suggestions-system/)  
[1269. Number of Ways to Stay in the Same Place After Some Steps](https://leetcode.com/contest/weekly-contest-164/problems/number-of-ways-to-stay-in-the-same-place-after-some-steps/)  