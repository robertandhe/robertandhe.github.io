---
layout: post
title: "Leetcode1024 - Video Stitching"
subtitle: 'Leetcode1024-dynamic programming'
date:       2019-12-04 14:46:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - dynamic programming
---

# 问题描述
You are given a series of video clips from a sporting event that lasted ``T`` seconds.  These video clips can be overlapping with each other and have varied lengths.

Each video clip ``clips[i]`` is an interval: it starts at time ``clips[i][0]`` and ends at time ``clips[i][1]``.  We can cut these clips into segments freely: for example, a clip ``[0, 7]`` can be cut into segments ``[0, 1] + [1, 3] + [3, 7]``.

Return the minimum number of clips needed so that we can cut the clips into segments that cover the entire sporting event (``[0, T]``).  If the task is impossible, return ``-1``.

### Example 1:
```c++
Input: clips = [[0,2],[4,6],[8,10],[1,9],[1,5],[5,9]], T = 10
Output: 3
Explanation: 
We take the clips [0,2], [8,10], [1,9]; a total of 3 clips.
Then, we can reconstruct the sporting event as follows:
We cut [1,9] into segments [1,2] + [2,8] + [8,9].
Now we have segments [0,2] + [2,8] + [8,10] which cover the sporting event [0, 10].
```
### Example 2:
```c++
Input: clips = [[0,1],[1,2]], T = 5
Output: -1
Explanation: 
We can't cover [0,5] with only [0,1] and [0,2].
```
### Example 3:
```c++
Input: clips = [[0,1],[6,8],[0,2],[5,6],[0,4],[0,3],[6,7],[1,3],[4,7],[1,4],[2,5],[2,6],[3,4],[4,5],[5,7],[6,9]], T = 9
Output: 3
Explanation: 
We can take clips [0,4], [4,7], and [6,9].
```
### Example 4:
```c++
Input: clips = [[0,4],[2,8]], T = 5
Output: 2
Explanation: 
Notice you can have extra video after the event ends.
```
### Note:
1. ``1 <= clips.length <= 100``
2. ``0 <= clips[i][0], clips[i][1] <= 100``
3. ``0 <= T <= 100``

# 解法
&emsp;这题给出一些视频片段，要求利用某些片段连成一段长度为T的视频，每个片段可以随意切分，求使用的最小片段数。  
&emsp;最值问题用什么？当然是动态规划了。这题的动态数组肯定是dp[i]表示到第i个片段使用的最小片段数。dp初始化为100，因为题目限定clips最多有100段视频。那么视频怎么连起来，肯定是先做时间在前面的，再做后面的，所以要按照视频片段播放时间点排序。状态转移方程就是对每个片段往前遍历所有片段，找到可以和它连起来的视频，假设往前遍历到j，dp[j]表示到clips[j][1]时间点使用的最小片段数，那么到clips[i][1]时间点使用的最小片段数就是dp[j] + 1了，同时clips[i]可能和前面超过一个片段可以连起来，所以应该去最小值。  
&emsp;如果到某个片段clips[i][1] >= T表示已经连到T时间段，且dp[i] != 100表示它已经用前面的连过（如clips={[0, 2], [4, 8]}, T=5, 不能连成5，返回-1）就可返回dp[i]。  
&emsp;同时为了更新初始时间点，开始时给clips推入{0, 0}，排序后到了0位置作为卡位。
```c++
int videoStitching(vector<vector<int>>& clips, int T) 
{
	clips.push_back({0, 0});
	sort(clips.begin(), clips.end());
	int res = -1, n = clips.size();
	vector<int> dp(n, 100);
	dp[0] = 0;
	for (int i = 1; i < n; ++i)
	{
		for (int j = i - 1; j >= 0; --j)
		{
			if (clips[j][1] >= clips[i][0]){
				dp[i] = min(dp[i], dp[j] + 1);
			}
		}
		if (clips[i][1] >= T && dp[i] != 100){
			res = dp[i];
			break;
		}
	}
	return res;
}
```
time complexity: O(n^2)  
space complexity: O(n)  
runtime: 100.00%  
memory usage: 100.00%  

# Link
[leetcode-1024](https://leetcode.com/problems/video-stitching/)