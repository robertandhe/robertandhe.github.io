---
layout: post
title: "Leetcode955-Delete Columns to Make Sorted II"
subtitle: 'Leetcode955-greedy'
date:       2019-10-25 17:42:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
---

# 问题描述
We are given an array ``A`` of ``N`` lowercase letter strings, all of the same length.

Now, we may choose any set of deletion indices, and for each string, we delete all the characters in those indices.

For example, if we have an array ``A = ["abcdef","uvwxyz"] ``and deletion indices ``{0, 2, 3}``, then the final array after deletions is ``["bef","vyz"]``.

Suppose we chose a set of deletion indices D such that after deletions, the final array has its elements in lexicographic order ``(A[0] <= A[1] <= A[2] ... <= A[A.length - 1])``.

Return the minimum possible value of D.length.

### example 1:
```c++
Input: ["ca","bb","ac"]
Output: 1
Explanation: 
After deleting the first column, A = ["a", "b", "c"].
Now A is in lexicographic order (ie. A[0] <= A[1] <= A[2]).
We require at least 1 deletion since initially A was not in lexicographic order, so the answer is 1.
```
### example 2:
```c++
Input: ["xc","yb","za"]
Output: 0
Explanation: 
A is already in lexicographic order, so we don't need to delete anything.
Note that the rows of A are not necessarily in lexicographic order:
ie. it is NOT necessarily true that (A[0][0] <= A[0][1] <= ...)
```
### example 3:
```c++
Input: ["zyx","wvu","tsr"]
Output: 3
Explanation: 
We have to delete every column.
```
### Note:
1. ``1 <= A.length <= 100``
2. ``1 <= A[i].length <= 100``

# 解法
又一道字典序问题，这道题给定一个字符串向量，要求给出删除多少个位置的字符后向量满足字典序。属于贪心算法的问题。因为假设当前字符串是满足字典序的，即后面字符串大于等于前面的字符串，此时在所有字符后面加上一个字符后还要满足字典序。如果当前就不满足字典序，后面再加字符也没有意义。即局部最优解等于全局最优解，满足贪心算法条件。为了存储当前满足字典序的字符串向量，额外创建一个向量，否则要存储应该要删除的下标，略显繁琐。
- 如果当前字符串大于前一个字符串，此时再加一个字符也还是满足字典序。
- 如果当前字符串等于前一个字符串，后面加的字符应该也满足字典序条件。
```c++
int minDeletionSize(vector<string>& A)
{
	int res = 0, n = A.size(), m = A[0].size();
	vector<string> B;
	for (int i = 0; i < n; ++i)
		B.push_back("");
	for (int i = 0; i < m; ++i)
	{
		bool isSorted = true;
		for (int j = 1; j < n; ++j)
		{
			if (B[j] == B[j - 1] && A[j][i] < A[j - 1][i]){
				isSorted = false;
				++res;
				break;
			}
		}
		if (isSorted){
			for (int j = 0; j < n; ++j)
				B[j] += A[j][i];
		}
	}
	return res;
}
```

时间复杂度：O(mn)  
空间复杂度：O(n)  
runtime : 67.12%   
memory usage: 100.00%

# link
[leetcode-955](https://leetcode.com/problems/delete-columns-to-make-sorted-ii/)