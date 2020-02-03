---
layout: post
title: "Leetcode1094-Car Pooling"
subtitle: 'Leetcode1094-greedy, priority_queue'
date:       2019-10-27 11:25:00
author: "mudux"
header-style: text
catalog: true
tags:
  - algorithm
  - leetcode
  - greedy
  - priority_queue
---

# 问题描述
You are driving a vehicle that has ``capacity`` empty seats initially available for passengers.  The vehicle ``only`` drives east (ie. it ``cannot`` turn around and drive west.)

Given a list of ``trips``, ``trip[i] = [num_passengers, start_location, end_location]`` contains information about the ``i``-th trip: the number of passengers that must be picked up, and the locations to pick them up and drop them off.  The locations are given as the number of kilometers due east from your vehicle's initial location.

Return ``true`` if and only if it is possible to pick up and drop off all passengers for all the given trips. 
### Example 1:
```c++
Input: trips = [[2,1,5],[3,3,7]], capacity = 4
Output: false
```
### Example 2:
```c++
Input: trips = [[2,1,5],[3,3,7]], capacity = 5
Output: true
```
### Example 3:
```c++
Input: trips = [[2,1,5],[3,5,7]], capacity = 3
Output: true
```
### Example 4:
```c++
Input: trips = [[3,2,7],[3,7,9],[8,3,9]], capacity = 11
Output: true
```
### Constraints:
1. ``trips.length <= 1000``
2. ``trips[i].length == 3``
3. ``1 <= trips[i][0] <= 100``
4. ``0 <= trips[i][1] < trips[i][2] <= 1000``
5. ``1 <= capacity <= 100000``
   
# 解法
## 解法 1
要求出能否走完全程。应该到每个要上车的位置求出能否载满，同时要减去在这要下车的人。为了求出到某个位置要下车的人，用一个priority_queue存储{end_location, num_passengers}的pair，用当前trip的start_location弹出heap中end_location小于当前trip的start_location的pair。为了使priority_queue中按照从小到大排，应该用greater cmp函数。同时原始trips可能是未排序的，应按照start_location排一下。
```c++
bool carPooling(vector<vector<int>>& trips, int capacity)
{
	int cnt = 0, n = trips.size();
	sort(trips.begin(), trips.end(), [](vector<int>& a, vector<int>& b) {return a[1] < b[1];});
	priority_queue<vector<int>, vector<vector<int>>, greater<vector<int>>> q;
	for (int i = 0; i < n; ++i)
	{
		vector<int> cur = trips[i];
		while (!q.empty() && cur[1] >= q.top()[0]){
			cnt -= q.top()[1];
			q.pop();	
		}
		q.push({cur[2], cur[0]});
		cnt += cur[0];
		if (cnt > capacity)
			return false;
	}
	return true;
}
```
时间复杂度：O(nlog(n))  
空间复杂度：O(n)  
runtime: 61.87%  
memory usage: 100.00%  
## 解法 2
这个解法利用了start_location和end_location在区间[1, 1000]内的限定条件，用长度为1001的向量seats表示对应里程的位置变动人数。然后遍历求和，求出每个里程车上总人数。有点精巧。
```c++
bool carPooling2(vector<vector<int>>& trips, int capacity)
{
	vector<int> seats(1001, 0);
	for (int i = 0; i < trips.size(); ++i)
	{
		seats[trips[i][1]] += trips[i][0];
		seats[trips[i][2]] -= trips[i][0];
	}
	if (seats[0] > capacity)
		return false;
	for (int i = 1; i < seats.size(); ++i)
	{
		seats[i] += seats[i - 1];
		if (seats[i] > capacity)
			return false;
	}
	return true;
}
```
时间复杂度：O(n + m)  
空间复杂度：O(m)  
runtime: 99.82%  
memory usage: 100.00%  


## 解法 3
今天兴致不错，谢谢排序玩玩，这里是插入排序。
```c++
void insertionSort(vector<vector<int>>& trips)
{
	int n = trips.size();
	for (int i = 1; i < n; ++i)
	{
		vector<int> tmp = trips[i];
		int j = i;
		for (; j > 0 && trips[j - 1][1] > tmp[1]; --j)
			trips[j] = trips[j - 1];
		trips[j] = tmp;
	}
}
```
时间复杂度： O(n^2)  
空间复杂度： O(1)  
runtime: 5.49%  
memory usage: 100.00%  
## 解法 4
这里是希尔排序。
```c++
void shellSort(vector<vector<int>>& trips)
{
	int n = trips.size();
	for (int increment = n / 2; increment > 0; increment /= 2)
	{
		for (int i = increment; i < n; ++i)
		{
			vector<int> tmp = trips[i];
			int j = i;
			for (; j >= increment; j -= increment)
			{
				if (trips[j - increment][1] > tmp[1]){
					trips[j] = trips[j - increment];
				}
				else 
					break;
			}
			trips[j] = tmp;
		}
	}
}
```
runtime: 11.81%  
memory usage: 100.00% 

## 解法 5
这里是堆排序。
```c++
#define LeftChild(i) (2 * i + 1)

void percDown(vector<vector<int>>& trips, int i, int N)
{
	int child;
	vector<int> tmp = trips[i];
	for (; LeftChild(i) < N; i = child)
	{
		child = LeftChild(i);
		if (child != N - 1 && trips[child + 1][1] > trips[child][1])
			++child;
		if (tmp[1] < trips[child][1])
			trips[i] = trips[child];
		else 
			break;
	}
	trips[i] = tmp;
}

void heapSort(vector<vector<int>>& trips)
{
	int n = trips.size();
	for (int i = n / 2; i >= 0; --i)
		percDown(trips, i, n);
	for (int i = n - 1; i > 0; --i)
	{
		swap(trips[0], trips[i]);
		percDown(trips, 0, i);
	}
}
```
runtime: 17.12%  
memory usage: 100.00% 

## 解法 6
这里是归并排序。
```c++
void Merge(vector<vector<int>>& trips, vector<vector<int>>& tmpArray, int leftPos, int rightPos, int rightEnd)
{
	int leftEnd = rightPos - 1, numElement = rightEnd - leftPos + 1;
	int tmpPos = leftPos;
	while (leftPos <= leftEnd && rightPos <= rightEnd){
		if (trips[leftPos][1] <= trips[rightPos][1]){
			tmpArray[tmpPos++] = trips[leftPos++];
		}
		else {
			tmpArray[tmpPos++] = trips[rightPos++];
		}
	}
	while (leftPos <= leftEnd)
		tmpArray[tmpPos++] = trips[leftPos++];
	while (rightPos <= rightEnd)
		tmpArray[tmpPos++] = trips[rightPos++];
	for (int i = 0; i < numElement; ++i, rightEnd--)
		trips[rightEnd] = tmpArray[rightEnd];
}

void MSort(vector<vector<int>>& trips, vector<vector<int>>& tmpArray, int left, int right)
{
	int center;
	if (left < right){
		center = left + (right - left) / 2;
		MSort(trips, tmpArray, left, center);
		MSort(trips, tmpArray, center + 1, right);
		Merge(trips, tmpArray, left, center + 1, right);
	}
}

void mergeSort(vector<vector<int>>& trips)
{
	int n = trips.size();
	vector<vector<int>> tmpArray(n, vector<int>(3, 0));
	MSort(trips, tmpArray, 0, n - 1);
}
```
runtime: 61.87%  
memory usage: 100.00% 

## 解法 7
这里是快排。
```c++
void insertSortQ(vector<vector<int>>& trips, int left, int right)
{
	for (int i = left + 1; i <= right; ++i)
	{
		vector<int> tmp = trips[i];
		int j = i;
		for (; j > left; --j)
		{
			if (trips[j - 1][1] > tmp[1])
				trips[j] = trips[j - 1];
			else 
				break;
		}
		trips[j] = tmp;
	}
}

vector<int> median3(vector<vector<int>>& trips, int left, int right)
{
	int center = left + (right - left) / 2;
	if (trips[left][1] > trips[center][1])
		swap(trips[left], trips[center]);
	if (trips[left][1] > trips[right][1])
		swap(trips[left], trips[right]);
	if (trips[center][1] > trips[right][1])
		swap(trips[center], trips[right]);
	swap(trips[center], trips[right - 1]);
	return trips[right - 1];
}

void QSort(vector<vector<int>>& trips, int left, int right)
{
	if (left + 3 < right){
		vector<int> pivot = median3(trips, left, right);
		int i = left, j = right - 1;
		for (;;)
		{
			while (trips[++i][1] < pivot[1])
				;
			while (trips[--j][1] > pivot[1])
				;
			if (i < j)
				swap(trips[i], trips[j]);
			else 
				break;
		}
		swap(trips[i], trips[right - 1]);
		QSort(trips, left, i - 1);
		QSort(trips, i + 1, right);
	}
	else {
		insertSortQ(trips, left, right);
	}
}

void quickSort(vector<vector<int>>& trips)
{
	int n = trips.size();
	QSort(trips, 0, n - 1);
}
```
runtime: 61.87%  
memory usage: 100.00% 

# link
[leetcode-1094](https://leetcode.com/problems/car-pooling/)  
[solution 2](https://www.acwing.com/solution/LeetCode/content/2562/)

