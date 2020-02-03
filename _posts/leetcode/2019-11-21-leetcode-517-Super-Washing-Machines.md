---
layout: post
title: "Leetcode517 - Super Washing Machines"
subtitle: 'Leetcode-dynamic programming'
date:       2019-11-21 15:33:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
You have **n** super washing machines on a line. Initially, each washing machine has some dresses or is empty.

For each **move**, you could choose **any m** (1 ≤ m ≤ n) washing machines, and pass **one dress** of each washing machine to one of its adjacent washing machines **at the same time**.

Given an integer array representing the number of dresses in each washing machine from left to right on the line, you should find the **minimum number of moves** to make all the washing machines have the same number of dresses. If it is not possible to do it, return -1.

### Example 1
```c++
Input: [1,0,5]

Output: 3

Explanation: 
1st move:    1     0 <-- 5    =>    1     1     4
2nd move:    1 <-- 1 <-- 4    =>    2     1     3    
3rd move:    2     1 <-- 3    =>    2     2     2  
```
### Example 2
```c++
Input: [0,3,0]

Output: 2

Explanation: 
1st move:    0 <-- 3     0    =>    1     2     0    
2nd move:    1     2 --> 0    =>    1     1     1    
```
### Example 3
```c++
Input: [0,2,0]

Output: -1

Explanation: 
It's impossible to make all the three washing machines have the same number of dresses.    
```
### Note:
1. The range of n is ``[1, 10000]``.
2. The range of dresses number in a super washing machine is ``[0, 1e5]``.

# 解法
这道题还是动态规划解法。首先求出所有dress总和看能否整除，不能整除就直接返回-1了，能整除就把平均值记为ave，这个就是move结束后所有洗衣机应该
有的dress数目。那么如何求move次数，这道题中有个特点可以选取m个洗衣机连续向左或者右移动一件dress，为了让它们平均肯定是从多的移到少的。用每台洗衣机中dress数目减去ave就是它需要移给别的获得得到的数量。例如machines = {0, 0, 11, 5}。ave=4, 差值数组为[-4, -4, 7, 1]分别表示第一台洗衣机需要获得4件衣服，第二台也要获得4件衣服，第三台给别的7件...。那么如何由这个求move次数，从左至右累加差值，第一次累加为[0, -8, 7, 1]， 第二次累加为[0, 0, -1, 1]，第三次累加为[0, 0, 0, 0]，move等于累加中所有累加和绝对值和所有原始差值中的最大值，这里即为8。  
那么为啥，因为当相邻的洗衣机一个需要加，一个需要减时是可以在一次move中解决的，只有都要加或者都要减才要累加move次数，所以需要求累加和绝对值极大值，那么为啥还要原始差值极大值，因为可能存在某个洗衣机要往两边move，如[0, 3, 0],累加和为0, -1, 1, 0，绝对值极大值为1,差值极大值为2，2才是结果。  
accumulate函数来自numeric头文件。
```c++
int findMinMoves(vector<int>& machines)
{
	int res = 0, n = machines.size();
	int sum = accumulate(machines.begin(), machines.end(), 0);
	if (sum % n != 0)
		return -1;
	int ave = sum / n, cnt = 0;
	for (int& m : machines)
	{
		cnt += m - ave;
		res = max(res, max(abs(cnt), m - ave));
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 6.52%  
memory usage: 100.00%   

# Link
[leetcode-517](https://leetcode.com/problems/super-washing-machines/)  
[solution-grandyang](https://www.cnblogs.com/grandyang/p/6648557.html)  
[solution](https://leetcode.com/problems/super-washing-machines/discuss/99185/super-short-easy-java-on-solution)