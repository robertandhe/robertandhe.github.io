---
layout: post
title: "Leetcode Weekly Contest 168"
subtitle: 'Leetcode Weekly Contest 168'
date:       2019-12-22 20:52:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - ordered map
  - sliding window
---

# 题目1 1295. Find Numbers with Even Number of Digits
## 题目描述
Given an array ``nums`` of integers, return how many of them contain an ``even`` number of digits.
#### Example 1:
```c++
Input: nums = [12,345,2,6,7896]
Output: 2
Explanation: 
12 contains 2 digits (even number of digits). 
345 contains 3 digits (odd number of digits). 
2 contains 1 digit (odd number of digits). 
6 contains 1 digit (odd number of digits). 
7896 contains 4 digits (even number of digits). 
Therefore only 12 and 7896 contain an even number of digits.
```
#### Example 2:
```c++
Input: nums = [555,901,482,1771]
Output: 1 
Explanation: 
Only 1771 contains an even number of digits.
```
#### Constraints:
- ``1 <= nums.length <= 500``
- ``1 <= nums[i] <= 10^5``

## 解法
&emsp;送分的。
```c++
int findNumbers(vector<int>& nums) 
{
	int res = 0;
	for (int& num : nums)
	{
		int cnt = 0;
		while (num){
			num /= 10;
			++cnt;
		}
		if (cnt % 2 == 0)
			++res;
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(1)  

# 题目2 1296. Divide Array in Sets of K Consecutive Numbers
## 题目描述
Given an array of integers ``nums`` and a positive integer ``k``, find whether it's possible to divide this array into sets of ``k`` consecutive numbers
Return ``True`` if its possible otherwise return ``False``.
#### Example 1:
```c++
Input: nums = [1,2,3,3,4,4,5,6], k = 4
Output: true
Explanation: Array can be divided into [1,2,3,4] and [3,4,5,6].
```
#### Example 2:
```c++
Input: nums = [3,2,1,2,3,4,3,4,5,9,10,11], k = 3
Output: true
Explanation: Array can be divided into [1,2,3] , [2,3,4] , [3,4,5] and [9,10,11].
```
#### Example 3:
```c++
Input: nums = [3,3,2,2,1,1], k = 3
Output: true
```
#### Example 4:
```c++
Input: nums = [1,2,3,4], k = 3
Output: false
Explanation: Each array should be divided in subarrays of size 3.
```

#### Constraints:
- ``1 <= nums.length <= 10^5``
- ``1 <= nums[i] <= 10^9``
- ``1 <= k <= nums.length``

## 解法
&emsp;这道题做的时候花了些时间，递归和滑动窗口竟然都超时了，后面用了有序哈希表才通过。
### 解法 1 
&emsp;滑动窗口,TLE
```c++
bool isPossibleDivide(vector<int>& nums, int k) 
{
	int n = nums.size();
	if (n % k != 0)
		return false;
	vector<bool> used(n, false);
	sort(nums.begin(), nums.end());
	for (int i = 0; i < n; ++i)
	{
		if (used[i])
			continue;
		used[i] =  true;
		int cnt = 1;
		int pos = i;
		while ((pos < n - 1) && (cnt < k)){
			++pos;
			while (pos < n - 1 && (nums[pos] == nums[i] + cnt - 1 || used[pos]))
				++pos;
			if (nums[pos] == nums[i] + cnt){
				++cnt;
				used[pos] = true;
			}
		}
		if (cnt == k)
			continue;
		else 
			return false;
	}
	return true;
}
```
### 解法 2 
&emsp;有序表。先用有序表存储每个值出现次数，在从开始循环，记录每次出现的pair的值num和出现次数cnt，如果有序表中有num+1, num + 2,...,num + k - 1的键值并且他们的次数大于等于cnt，则满足能够组成连续序列，并把它们的值减cnt。如果不满足就可以返回false。
```c++
bool isPossibleDivide(vector<int>& nums, int k) 
{
	int n = nums.size();
	if (n % k != 0)
		return false;
	vector<bool> used(n, false);
	map<int, int> mem;
	for (int& num : nums)
		mem[num] += 1;
	for (auto& a : mem)
	{
		int num = a.first;
		int cnt = a.second;
		if (cnt == 0)
			continue;
		for (int i = 1; i <= k - 1; ++i)
		{
			if (mem.count(num + i) && mem[num + i] >= cnt){
				mem[num + i] -= cnt;
			}
			else 
				return false;
		}
	}
	return true;
}
```
time complexity: O(n)  
space complexity: O(n)  

# 题目3 1297. Maximum Number of Occurrences of a Substring
## 题目描述
Given a string ``s``, return the maximum number of ocurrences of any substring under the following rules:
- The number of unique characters in the substring must be less than or equal to ``maxLetters``.
- The substring size must be between ``minSize`` and ``maxSize`` inclusive.
#### Example 1:
```c++
Input: s = "aababcaab", maxLetters = 2, minSize = 3, maxSize = 4
Output: 2
Explanation: Substring "aab" has 2 ocurrences in the original string.
It satisfies the conditions, 2 unique letters and size 3 (between minSize and maxSize).
```
#### Example 2:
```c++
Input: s = "aaaa", maxLetters = 1, minSize = 3, maxSize = 3
Output: 2
Explanation: Substring "aaa" occur 2 times in the string. It can overlap.
```
#### Example 3:
```c++
Input: s = "aabcabcab", maxLetters = 2, minSize = 2, maxSize = 3
Output: 3
```
#### Example 4:
```c++
Input: s = "abcde", maxLetters = 2, minSize = 3, maxSize = 3
Output: 0
```

#### Constraints:
- ``1 <= s.length <= 10^5``
- ``1 <= maxLetters <= 26``
- ``1 <= minSize <= maxSize <= min(26, s.length)``
- ``s`` only contains lowercase English letters.

## 解法
&emsp;用滑动窗口来解。首先应该发现minSize才是我们需要的字符串长度，因为如果更长的字符串满足条件，它的子串肯定只会出现次数更多，所以仅需考虑所有长度为minSize的子字符串，在他们中间还要满足不同字符个数小于等于maxLetters。  
&emsp;用count数组来统计子串中所有字符个数，如果长度大于minSize了就需要滑动了，如果start位置字符加1后为0，说明子串中没有这个字符了，应该erase。如果字符串长度为minSize且count大小小于等于maxLetters就说明满足条件，此时occurrence对应字符串个数加1，更新res。
```c++
int maxFreq(string s, int maxLetters, int minSize, int maxSize) 
{
	int start = 0, res = 0;
	unordered_map<int, int> count;
	unordered_map<string, int> occurence;
	for (int i = 0; i < s.length(); ++i)
	{
		count[s[i]]++;
		if (i - start + 1 > minSize){
			if (--count[s[start]] == 0)
				count.erase(s[start]);
			++start;
		}
		if (i - start + 1 == minSize && count.size() <= maxLetters)
			res = max(res, ++occurence[s.substr(start, minSize)]);
	}
	return res;
}
```
time complexity: O(n)  
space complexity: O(n)  

# References
[leetcode-1295](hhttps://leetcode.com/contest/weekly-contest-168/problems/find-numbers-with-even-number-of-digits/)  
[leetcode-1296](https://leetcode.com/contest/weekly-contest-168/problems/divide-array-in-sets-of-k-consecutive-numbers/)  
[leetcode-1297](https://leetcode.com/contest/weekly-contest-168/problems/maximum-number-of-occurrences-of-a-substring/)  