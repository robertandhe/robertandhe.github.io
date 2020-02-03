---
layout: post
title: "Leetcode376-Wiggle Subsequence"
subtitle: 'Leetcode376-stack,dp'
date:       2019-10-16 20:02:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - greedy
  - dynamic programming
---
# 问题描述
A sequence of numbers is called a wiggle sequence if the differences between successive numbers strictly alternate between positive and negative. The first difference (if one exists) may be either positive or negative. A sequence with fewer than two elements is trivially a wiggle sequence. 

For example, [1,7,4,9,2,5] is a wiggle sequence because the differences (6,-3,5,-7,3) are alternately positive and negative. In contrast, [1,4,7,2,5] and [1,7,4,5,5] are not wiggle sequences, the first because its first two differences are positive and the second because its last difference is zero.

Given a sequence of integers, return the length of the longest subsequence that is a wiggle sequence. A subsequence is obtained by deleting some number of elements (eventually, also zero) from the original sequence, leaving the remaining elements in their original order.
Example 1:
```c++
Input: [1,7,4,9,2,5]
Output: 6
Explanation: The entire sequence is a wiggle sequence.
```
Example 2:
```c++
Input: [1,17,5,10,13,15,10,5,16,8]
Output: 7
Explanation: There are several subsequences that achieve this length. One is [1,17,10,13,10,16,8].
```
Example 3:
```c++
Input: [1,2,3,4,5,6,7,8,9]
Output: 2
```
Follow up:
Can you do it in O(n) time?

# 解法
## 解法1
这种求最长的问题基本都是动态规划或者贪心算法。在这道题中，要求wiggle sequence,即sequence元素间差值符号正负相间，如果有连续差值
为同一符号，则应跳过。在我的解法中，用一个vector来存储sequence，如果现在要求差值为负，接下来元素和前一个位置差值刚好为负，则当前元素符合条件，存入向量，如果差值还是为正，则说明当前元素比上个元素和上上个元素都大，这种情况下就用当前元素替换vector.back(),因为这样会给后续的下降更多余量，如果要求差值为正，则情况相反。  
然后边界条件处理，原始向量长度小于2，起始符号处理。
```c++
int wiggleMaxLength(vector<int>& nums)
{
	if (nums.size() <= 1)
		return nums.size();
	bool ascend = true;
	vector<int> vec;
	for (int& num : nums)
	{
		if (vec.empty()){
			vec.push_back(num);
			continue;
		}
		if (vec.size() == 1 && num != vec.back()){
			vec.push_back(num);
			if (vec[1] > vec[0])
				ascend = false;
			continue;
		}
		if (ascend){
			if (num > vec.back()){
				vec.push_back(num);
				ascend = false;
			}
			else
				vec.back() = num;
		}
		else {
			if (num < vec.back()){
				vec.push_back(num);
				ascend = true;
			}
			else
				vec.back() = num;
		}
	}
	return vec.size();
}
```
时间复杂度：O(n)  
空间复杂度：O(n)
## 解法2
这种动态规划解法中，用p[i]表示到i位置且上一差值为正的最大sequence长度，q[i]表示到i位置且上一差值为负的最大sequence长度。
对nums中每个位置元素，和它之前所有位置j元素比较，如果nums[i]大于nums[j],则更新p[i],此时的p[i]应为当前p[i]和到j位置且上一差值
为负的最大sequence长度加1中的最大值，略微绕口，看代码``p[i] = max(p[i], q[j] + 1)``。如果nums[i]小于nums[j]，则更新q[i]。如果nums[i]等于nums[j]，不处理。  
边界条件，``nums.size() <= 1``处理。
```c++
int wiggleMaxLength(vector<int>& nums)
{
	if (nums.size() <= 1)
		return nums.size();
	vector<int> p(nums.size(), 1);
	vector<int> q(nums.size(), 1);
	for (int i = 1; i < nums.size(); ++i)
	{
		for (int j = 0; j < i; ++j)
		{
			if (nums[i] > nums[j])
				p[i] = max(p[i], q[j] + 1);
			else if (nums[i] < nums[j])
				q[i] = max(q[i], p[j] + 1);
		}
	}
	return max(p.back(), q.back());
}
```
时间复杂度：O(n^2)  
空间复杂度：O(n)
## 解法3
贪心算法，和解法二相同，在解法二中，真正有用的是上一p[j]和q[j]。这种接法有两个变量p和q表示上次差值分别为正和负的sequence最大长度。
当前位置元素大于上一位置元素，当前p应该是上一q加1，当前位置元素小于上一位置元素，更新当前q。  
边界条件处理，``min(n, max(p, q))``。
```c++
int wiggleMaxLength(vector<int>& nums)
{
	int p = 1, q = 1, n = nums.size();
	for (int i = 1; i < nums.size(); ++i)
	{
		if (nums[i] > nums[i - 1])
			p = q + 1;
		else if (nums[i] < nums[i - 1])
			q = p + 1;
	}
	return min(n, max(p, q));
}
```
时间复杂度：O(n)  
空间复杂度：O(1)

# 总结
解法2和解法3基本和我的思想相同，但是代码质量还是有差距。解法2用``p[i] = max(p[i], q[j] + 1)``更新当前sequence结尾，解法3用``p = q + 1``更新，如果出现单调递增序列，p维持不变。和我的``vec.back() = num``思想一样。

# link
[leetcode-376](https://leetcode.com/problems/wiggle-subsequence/)
[solution2 & solution3](https://www.cnblogs.com/grandyang/p/5697621.html)