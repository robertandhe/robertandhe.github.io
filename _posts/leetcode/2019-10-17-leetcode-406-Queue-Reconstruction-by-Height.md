---
layout: post
title: "Leetcode406-Queue Reconstruction by Height"
subtitle: 'Leetcode406-stack'
date:       2019-10-17 10:30:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
---

# 问题描述
Suppose you have a random list of people standing in a queue. Each person is described by a pair of integers (h, k), where h is the height of the person and k is the number of people in front of this person who have a height greater than or equal to h. Write an algorithm to reconstruct the queue.
Note:  
The number of people is less than 1,100.
Example:
```c++
Input:
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]

Output:
[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]
```

# 解法
## 解法1
首先确定最小元素a，因为根据a[1]能确定a在数组中的位置。同时对于同样大小的元素a和b，应该按照a[1],b[1]升序排列，因为同样大小也算在计数里面。
在这里用priority_queue来排序，先弹出第一个，即最小值a。根据a[1]确定在结果res中的位置，确定标准是统计到某个位置大于a[0]或者为空的个数是否等于a[1]。这里初始化res时用[-1, -1]，按道理身高肯定大于0，但测试集中还有身高为0的！所以用-1表示这个位置还没占。最后统计前面符号条件的个数后，还要向后找到下一个空的位置放入当前最小值。
```c++
vector<vector<int>> reconstructQueue(vector<vector<int>>& people)
{
	int n = people.size();
	vector<vector<int>> res(n, vector<int>(2, -1));
	auto cmp = [](vector<int>& a, vector<int>& b) {if (a[0] == b[0]) return a[1] > b[1]; return a[0] > b[0];};
	priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> q(cmp);
	for (auto& person : people)
		q.push(person);
	while (!q.empty()){
		int cnt = 0;
		vector<int> cur = q.top();
		q.pop();
		int i = 0;
		while (i < n && cnt < cur[1]){
			if (res[i][0] == -1 || res[i][0] >= cur[0]){
				++cnt;
			}
			++i;
		}
		while(res[i][0] != -1)
			++i;
		res[i] = cur;
	}
	return res;
}
```
时间复杂度：O(n^2)  
空间复杂度：O(n)
## 解法2
和解法1思想一样，只是用STL sort来给数组排序。
```c++
vector<vector<int>> reconstructQueue(vector<vector<int>>& people)
{
	int n = people.size();
	vector<vector<int>> res(n, vector<int>(2, -1));
	auto cmp = [](vector<int>& a, vector<int>& b) {if (a[0] == b[0]) return a[1] < b[1]; return a[0] < b[0];};
	sort(people.begin(), people.end(), cmp);
	for (auto& cur : people)
	{
		int cnt = 0, i = 0;
		while (cnt < cur[1]){
			if (res[i][0] == -1 || res[i][0] >= cur[0])
				++cnt;
			++i;
		}
		while (i < n && res[i][0] != -1)
			++i;
		res[i] = cur;
	}
	return res;
}
```
时间复杂度：O(n^2)  
空间复杂度：O(n)
## 解法3
先把数组排序，大的在前，同样大小的1位置小的在前，建立空数组，遍历people按照1位置插入。关键在于数组是按照从大到小排列的，
当前元素1位置表示它之前有多少个大于等于它的元素，先按照1位置插入，后面来的小的元素不会破坏前面的次序。
```c++
vector<vector<int>> reconstructQueue(vector<vector<int>>& people)
{
	vector<vector<int>> res;
	sort(people.begin(), people.end(), [](vector<int>& a, vector<int>& b) {if (a[0] == b[0]) return a[1] < b[1]; return a[0] > b[0];});
	for (auto& person : people)
	{
		res.insert(res.begin() + person[1], person);
	}	
	return res;
}
```
时间复杂度：O(nlogn)  
空间复杂度：O(n)
## 解法4
解法3要建立新数组，这里就地排序。先按照解法3同样排序，对当前元素a统计前面到那个位置大于等于它的个数等于a[1]。这个位置就是要交换的位置，在这个位置后所有元素后移1位。
```c++
vector<vector<int>> reconstructQueue(vector<vector<int>>& people)
{
	vector<vector<int>> res;
	sort(people.begin(), people.end(), [](vector<int>& a, vector<int>& b) {if (a[0] == b[0]) return a[1] < b[1]; return a[0] > b[0];});
	for (auto& person : people)
	{
		res.insert(res.begin() + person[1], person);
	}	
	return res;
}
```
时间复杂度：O(n^2)  
空间复杂度：O(1)

# link
[leetcode-406](https://leetcode.com/problems/queue-reconstruction-by-height/)  
[solution3 & solution4](https://www.cnblogs.com/grandyang/p/5928417.html)