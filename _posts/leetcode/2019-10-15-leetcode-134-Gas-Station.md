---
layout: post
title: "Leetcode134-Gas Station"
subtitle: 'Leetcode134-greedy'
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
---

# 问题描述
There are N gas stations along a circular route, where the amount of gas at station i is gas[i].  
You have a car with an unlimited gas tank and it costs cost[i] of gas to travel from station i to its next station (i+1). You begin the journey with an empty tank at one of the gas stations.  
Return the starting gas station's index if you can travel around the circuit once in the clockwise direction, otherwise return -1.
Note:
- If there exists a solution, it is guaranteed to be unique.
- Both input arrays are non-empty and have the same length.
- Each element in the input arrays is a non-negative integer.

Example1:
```c++
Input: 
gas  = [1,2,3,4,5]
cost = [3,4,5,1,2]

Output: 3

Explanation:
Start at station 3 (index 3) and fill up with 4 unit of gas. Your tank = 0 + 4 = 4
Travel to station 4. Your tank = 4 - 1 + 5 = 8
Travel to station 0. Your tank = 8 - 2 + 1 = 7
Travel to station 1. Your tank = 7 - 3 + 2 = 6
Travel to station 2. Your tank = 6 - 4 + 3 = 5
Travel to station 3. The cost is 5. Your gas is just enough to travel back to station 3.
Therefore, return 3 as the starting index.
```

Example2:
```c++
Input: 
gas  = [2,3,4]
cost = [3,4,3]

Output: -1

Explanation:
You can't start at station 0 or 1, as there is not enough gas to travel to the next station.
Let's start at station 2 and fill up with 4 unit of gas. Your tank = 0 + 4 = 4
Travel to station 0. Your tank = 4 - 3 + 2 = 3
Travel to station 1. Your tank = 3 - 3 + 3 = 3
You cannot travel back to station 2, as it requires 4 unit of gas but you only have 3.
Therefore, you can't travel around the circuit once no matter where you start.
```

# 解法
## 解法1
这道题总体来说还是比较简单直观，首先用暴力法，从头遍历，每一个位置轮流当作起点，从起点开始在每个加油站算gas还剩多少，如果到某个位置gas小于0了，说明从这个起点开始没法走完全程，则退出从下一起点重新开始，如果某一起点能够走完全程，则返回。
```c++
int canCompleteCircuit(vector<int>& gas, vector<int>& cost) 
{
	int n = gas.size();
	for (int i = 0; i < n; ++i)	
	{
		int cnt = 0, tank = 0;
		for (int j = i; cnt < n; ++j, ++cnt)
		{
			int t = j % n;
			tank += gas[t] - cost[t];
			if (tank < 0)
				break;
		}
		if (cnt == n)
			return i;
	}
	return -1;
}
```

## 解法2
想要走完全程必须满足gas总量大于cost总量。先设置start为0，从start到当前位置gas余量用sum表示。非常重要的一点是，如果到某一点sum小于0，则start置为i+1，因为sum小于0表示从当前起点开始无法到达当前位置，而起点gas余量肯定大于等于0，相当于start对sum有贡献，start后一个gas station肯定不能当做新的起点，以此类推，中间所有位置都不能当做新的起点，所以start=i+1。而sum小于0更新保证从新起点开始sum不受之前影响，
和max sum思想异曲同工。
```c++
int canCompleteCircuit(vector<int>& gas, vector<int>& cost) 
{
	int total = 0, sum = 0, start = 0;
	for (int i = 0; i < gas.size(); ++i)
	{
		total += gas[i] - cost[i];
		sum += gas[i] - cost[i];
		if (sum < 0){
			sum = 0;
			start = i + 1;
		}
	}
	return total < 0 ? -1 : start;
}
```

## 解法3
这个方法从后往前推，用mx记录gas余量最大值，如果当前位置gas余量total大于mx，说明以当前位置为起点更合适，更加宽松，所以更新start和mx。
```c++
int canCompleteCircuit(vector<int>& gas, vector<int>& cost) 
{
	int start = 0, mx = -1, total = 0;
	for (int i = gas.size() - 1; i >= 0; --i)
	{
		total += gas[i] - cost[i];
		if (total > mx){
			mx = total;
			start = i;
		}
	}
	return total < 0 ? -1 : start;
}
```

# link
[leetcode-134-gas-station](https://leetcode.com/problems/gas-station/)