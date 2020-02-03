---
layout: post
title: "Leetcode870-Advantage Shuffle"
subtitle: 'Leetcode870-greedy,binary search,heap'
date:       2019-10-26 20:15:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
  - priority-queue
  - binary search
---

# 问题描述
Given two arrays ``A`` and ``B`` of equal size, the advantage of ``A`` with respect to ``B`` is the number of indices ``i`` for which ``A[i] > B[i]``.

Return ``any`` permutation of ``A`` that maximizes its advantage with respect to ``B``.
### Example 1:
```c++
Input: A = [2,7,11,15], B = [1,10,4,11]
Output: [2,11,7,15]
```
### Example 2:
```c++
Input: A = [12,24,8,32], B = [13,25,32,11]
Output: [24,32,8,12]
```
### Note:
1. ``1 <= A.length = B.length <= 10000``
2. ``0 <= A[i] <= 10^9``
3. ``0 <= B[i] <= 10^9``

# 解法
## 解法1
重排A使A对应位置大于B的个数最多，思路就是用A中刚好大于B中元素的来对应，如果没有大于B的，就用A中最小来对应，为贪心算法。  
要使A中元素有序，且易找到刚好大于B中的位置，这里用multiset存储A，直接STL upper_bound即可找到刚好大于的位置，如果没有就用最小来对应。
```c++
vector<int> advantageCount(vector<int>& A, vector<int>& B)
{
	vector<int> res;
	multiset<int> s(A.begin(), A.end());
	for (int i = 0; i < B.size(); ++i)
	{
		auto it = *s.rbegin() <= B[i] ? s.begin() : s.upper_bound(B[i]);
		res.push_back(*it);
		s.erase(it);
	}
	return res;
}
```
时间复杂度：O(n^2log(n))  
空间复杂度：O(n)  
runtime: 52.71%  
memory usage: 16.67%
## 解法2
思路和解法1一样，写个二分玩玩。
```c++
vector<int> advantageCount(vector<int>& A, vector<int>& B)
{
	vector<int> res;
	vector<int> s(A.begin(), A.end());
	sort(s.begin(), s.end());
	for (int i = 0; i < B.size(); ++i)
	{
		int it = 0;
		if (s.back() <= B[i]){
			it = 0;
		}
		else {
			int left = 0, right = s.size();
			while (left < right){
				int mid = left + (right - left) / 2;
				if (s[mid] <= B[i]){
					left = mid + 1;
				}
				else {
					right = mid;
				}
			}
			it = left;
		}
		res.push_back(s[it]);
		s.erase(s.begin() + it);
	}
	return res;
}
```
时间复杂度：O(n^2log(n))  
空间复杂度：O(n)  
runtime: 10.00%  
memory usage: 91.67%
## 解法3
使B中元素也有序，但给B排序是会打乱位置，用priority_queue存储{value, position}的pair，默认按照value less排序，但priority_queue大的在上。
使A中也有序，用两个指针指向left，right，对B中当前最大，如果A中right对应最大小于它，就用left指向最小来对应，否则就用right指向位置。
```c++
vector<int> advantageCount3(vector<int>& A, vector<int>& B)
{
	int n = A.size();
	vector<int> res(n, 0);
	priority_queue<pair<int, int>> q;
	sort(A.begin(), A.end());
	int left = 0, right = n - 1;
	for (int i = 0; i < n; ++i)
		q.push({B[i], i});
	while (!q.empty()){
		int val = q.top().first, idx = q.top().second;
		q.pop();
		if (A[right] > val)
			res[idx] = A[right--];
		else
			res[idx] = A[left++];
	}
	return res;
}
```
runtime: 43.86%  
memory usage: 91.67%

# link
[leetcode-870](https://leetcode.com/problems/advantage-shuffle/)  
[solution](https://www.cnblogs.com/grandyang/p/10759525.html)  
[二分法小结](https://www.cnblogs.com/grandyang/p/6854825.html)