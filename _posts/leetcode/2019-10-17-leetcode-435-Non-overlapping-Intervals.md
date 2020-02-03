---
layout: post
title: "Leetcode435-Non-overlapping-intervals"
subtitle: 'Leetcode435-greedy'
date:       2019-10-17 16:30:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
---

# 问题描述
Given a collection of intervals, find the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping.   
Example 1:
```c++
Input: [[1,2],[2,3],[3,4],[1,3]]
Output: 1
Explanation: [1,3] can be removed and the rest of intervals are non-overlapping.
```
Example 2:
```c++
Input: [[1,2],[1,2],[1,2]]
Output: 2
Explanation: You need to remove two [1,2] to make the rest of intervals non-overlapping.
```
Example 3:
```c++
Input: [[1,2],[2,3]]
Output: 0
Explanation: You don't need to remove any of the intervals since they're already non-overlapping.
```
Note: 
- You may assume the interval's end point is always bigger than its start point.
- Intervals like [1,2] and [2,3] have borders "touching" but they don't overlap each other.
  
# 解法
## 解法1
挺简单的贪心问题，首先应把数组排序，直接STL sort，实在闲的慌可以自己写sort。sort的标准是0位置小的在前，0位置相同则1位置小的在前。
然后开始遍历，这里用栈存储上一个位置元素，如果当前位置元素0位置和上一个相同，则res加1，因为1位置肯定大于等于上一个元素1位置，形成对
后面的限制，应舍去。如果当前位置元素0位置大于等于前一元素1位置，则两个区间是断开的，符合条件，存入堆栈，继续遍历。到最后当前元素0位置肯定是小于前一元素1位置了，这是应看那个1位置小就留那个，因为这会给后面元素更大余量，更新堆栈，继续遍历。  
边界条件处理。
```c++
int eraseOverlapIntervals(vector<vector<int>>& intervals) 
{
	int res = 0;
	if (intervals.empty())
		return res;
	stack<vector<int>> st;
	sort(intervals.begin(), intervals.end(), [](vector<int>& a, vector<int>& b) {if (a[0] == b[0]) return a[1] < b[1]; return a[0] < b[0];});
	st.push(intervals[0]);
	for (int i = 1; i < intervals.size(); ++i)
	{
		if (intervals[i][0] == st.top()[0]){
			++res;
			continue;
		}
		if (intervals[i][0] >= st.top()[1]){
			st.push(intervals[i]);
			continue;
		}	
		if (intervals[i][1] <= st.top()[1]){
			st.pop();
			st.push(intervals[i]);
			++res;
		}
		else 
			++res;
	}
	return res;
}
```
时间复杂度：O(nlogn)  
空间复杂度：O(n)
## 解法2
解法1用堆栈存储上一个元素，其实只需要知道上一个元素位置就行了，减小内存开销。
```c++
int eraseOverlapIntervals(vector<vector<int>>& intervals) 
{
	int res = 0;
	if (intervals.empty())
		return res;
	stack<vector<int>> st;
	sort(intervals.begin(), intervals.end(), [](vector<int>& a, vector<int>& b) {if (a[0] == b[0]) return a[1] < b[1]; return a[0] < b[0];});
	int pre = 0;
	for (int i = 1; i < intervals.size(); ++i)
	{
		if (intervals[i][0] == intervals[pre][0]){
			++res;
			continue;
		}
		if (intervals[i][0] >= intervals[pre][1]){
			pre = i;
			continue;
		}
		if (intervals[i][1] <= intervals[pre][1]){
			pre = i;
			++res;
		}
		else 
			++res;
	}
	return res;
}
```
时间复杂度：O(nlogn)  
空间复杂度：O(1)  
虽然没有堆栈开销，但oJ时时间空间效率并没有提升。  
## 解法3
和解法2一样，代码更优美。
```c++
int eraseOverlapIntervals(vector<vector<int>>& intervals) 
{
	int res = 0, last = 0, n = intervals.size();
	sort(intervals.begin(), intervals.end());
	for (int i = 1; i < n; ++i)
	{
		if (intervals[i][0] < intervals[last][1]){
			++res;
			if (intervals[i][1] < intervals[last][1])
				last = i;
		}
		else 
			last = i;
	}
	return res;
}
```
时间复杂度：O(nlogn)  
空间复杂度：O(1)  

# link
[leetcode-435](https://leetcode.com/problems/non-overlapping-intervals/)  
[solution 3](https://www.cnblogs.com/grandyang/p/6017505.html)