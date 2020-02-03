---
layout: post
title: "Leetcode1005-Maximize Sum Of Array After K Negations"
subtitle: 'Leetcode-greedy'
date:       2019-10-28 11:29:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
---

# 问题描述
Given an array ``A`` of integers, we ``must`` modify the array in the following way: we choose an ``i`` and replace`` A[i]`` with ``-A[i]``, and we repeat this process ``K`` times in total.  (We may choose the same index ``i`` multiple times.)

Return the largest possible sum of the array after modifying it in this way.

### Example 1:
```c++
Input: A = [4,2,3], K = 1
Output: 5
Explanation: Choose indices (1,) and A becomes [4,-2,3].
```
### Example 2:
```c++
Input: A = [3,-1,0,2], K = 3
Output: 6
Explanation: Choose indices (1, 2, 2) and A becomes [3,1,0,2].
```
### Example 3:
```c++
Input: A = [2,-3,-1,5,-4], K = 2
Output: 13
Explanation: Choose indices (1, 4) and A becomes [2,3,-1,5,4].
```
### Note:
1. ``1 <= A.length <= 10000``
2. ``1 <= K <= 10000``
3. ``-100 <= A[i] <= 100``

# 解法
## 解法 1
肯定优先翻转负数，所以要先排序。负数翻转完就要翻转正数了，如果此时``K%2==0``，那么正数翻转偶数次，相当于啥都不做。如果``K%2==1``,就要调
一个正数翻转，应该挑这个正数和前一个非正数绝对值小的翻转。如果前一个非正数绝对值小，那么应该翻转它翻转之后的正数，因为之前已经加过一次正数，就要
加上``2 * A[i - 1]``,在加``A[i]``。否则加``-A[i]``。
```c++
int largestSumAfterKNegations(vector<int>& A, int K)
{
	int res = 0;
	sort(A.begin(), A.end());
	for (int i = 0; i < A.size(); ++i)
	{
		if (A[i] < 0 && K){
			--K;
			res += -A[i];
		}
		else if (K % 2){
			if (i > 0 && abs(A[i - 1]) < abs(A[i])){
				res += 2 * A[i - 1];
				res += A[i];
			}
			else 
				res += -A[i];
			K = 0;
		}
		else
			res += A[i];
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  
runtime: 68.31%  
memory usage: 50.00%  

# 解法 2
利用最小堆，每次挑最小的翻转，干净利落。
```c++
int largestSumAfterKNegations2(vector<int>& A, int K)
{
	int res = 0;
	priority_queue<int, vector<int>, greater<int>> q(A.begin(), A.end());
	while (K){
		int t = q.top();
		q.pop();
		q.push(-t);
		--K;
	}
	while (!q.empty()){
		res += q.top();
		q.pop();
	}
	return res;
}
```
time complexity: O(nlong)  
space complexity: O(n)  
runtime: 24.21%  
memory usage: 85.71%  


<html>
	<body>
		<p style="color:#9963ff" size=4 face="宋体";>
			至于为啥解法2比解法1空间占优，暂时唔知啊！！！
		</p>
	</body>
</html>



# link
[leetcode-1005](https://leetcode.com/problems/maximize-sum-of-array-after-k-negations/)  