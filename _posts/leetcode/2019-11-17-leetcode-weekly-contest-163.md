---
layout: post
title: "Leetcode-Weekly Contest 163"
subtitle: 'Leetcode-array, dynamic programming, BFS, tree'
date:       2019-11-17 19:02:00
author: "mudux"
header-style: text
catalog: True
tags:
  - algorithm
  - leetcode
  - array
  - tree
  - dynamic programming
  - BFS
---


# 来源
第一次参加weekly contest,共四道题目，排名一般。

# 第一道 1260. Shift 2D Grid
## 问题描述
Given a 2D ``grid`` of size ``n * m`` and an integer`` k``. You need to shift the ``grid k ``times.   
In one shift operation:
- Element at ``grid[i][j]`` becomes at ``grid[i][j + 1]``.
- Element at ``grid[i][m - 1]`` becomes at ``grid[i + 1][0]``.
- Element at ``grid[n - 1][m - 1]`` becomes at ``grid[0][0]``.

Return the 2D grid after applying shift operation ``k`` times.
#### Example 1:
```c++
Input: grid = [[1,2,3],[4,5,6],[7,8,9]], k = 1
Output: [[9,1,2],[3,4,5],[6,7,8]]
```
#### Example 2:
```c++
Input: grid = [[3,8,1,9],[19,7,2,5],[4,6,11,10],[12,0,21,13]], k = 4
Output: [[12,0,21,13],[3,8,1,9],[19,7,2,5],[4,6,11,10]]
```
#### Example 3:
```c++
Input: grid = [[1,2,3],[4,5,6],[7,8,9]], k = 9
Output: [[1,2,3],[4,5,6],[7,8,9]]
```
#### Constraints:
- ``1 <= grid.length <= 50``
- ``1 <= grid[i].length <= 50``
- ``-1000 <= grid[i][j] <= 1000``
- ``0 <= k <= 100``

## 解法
挺简单的一道题，就是二维数组内元素往右移位，超出一行就移到下一行开头。建立另一二维数组存储结果，首先需要统计元素个数numElement，因为超出了要从左上角开始。k要先取余numElement的余数，移位过程也要余numElement。
```c++
vector<vector<int>> shiftGrid(vector<vector<int>>& grid, int k)
{
	int m = grid.size(), n = grid[0].size(), numElements = m * n;
	k %= numElements;
	vector<vector<int>> res(m, vector<int>(n));
	for (int i = 0; i < m; ++i)
	{
		for (int j = 0; j < n; ++j)
		{
			int t = i * n + j;
			t += k;
			t %= numElements;
			int r = t / n, c = t % n;
			res[r][c] = grid[i][j];
		}
	}
	return res;
}
```


# 第二道 1261. Find Elements in a Contaminated Binary Tree
## 问题描述
Given a binary tree with the following rules:
1. ``root.val == 0``
2. If ``treeNode.val == x`` and ``treeNode.left != null``, then ``treeNode.left.val == 2 * x + 1``
3. If ``treeNode.val == x`` and ``treeNode.right != null``, then ``treeNode.right.val == 2 * x + 2``

Now the binary tree is contaminated, which means all ``treeNode.val`` have been changed to ``-1``.  
You need to first recover the binary tree and then implement the ``FindElements`` class:
- ``FindElements(TreeNode* root)`` Initializes the object with a contamined binary tree, you need to recover it first.
- ``bool find(int target)`` Return if the ``target`` value exists in the recovered binary tree.

#### Example 1:
![example1](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-17/1261-example1.PNG)
```c++
Input
["FindElements","find","find"]
[[[-1,null,-1]],[1],[2]]
Output
[null,false,true]
Explanation
FindElements findElements = new FindElements([-1,null,-1]); 
findElements.find(1); // return False 
findElements.find(2); // return True 
```
#### Example 2:
![example2](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-17/1261-example2.PNG)
```c++
Input
["FindElements","find","find","find"]
[[[-1,-1,-1,-1,-1]],[1],[3],[5]]
Output
[null,true,true,false]
Explanation
FindElements findElements = new FindElements([-1,-1,-1,-1,-1]);
findElements.find(1); // return True
findElements.find(3); // return True
findElements.find(5); // return False
```
#### Example 3:
![example3](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-17/1261-example3.PNG)
```c++
Input
["FindElements","find","find","find","find"]
[[[-1,null,-1,-1,null,-1]],[2],[3],[4],[5]]
Output
[null,true,false,false,true]
Explanation
FindElements findElements = new FindElements([-1,null,-1,-1,null,-1]);
findElements.find(2); // return True
findElements.find(3); // return False
findElements.find(4); // return False
findElements.find(5); // return True
```

#### Constraints:
- ``TreeNode.val == -1``
- The height of the binary tree is less than or equal to ``20``
- The total number of nodes is between ``[1, 10^4]``
- Total calls of ``find()`` is between ``[1, 10^4]``
- ``0 <= target <= 10^6``

## 解法
挺简单一题，FindElements遍历二叉树，恢复被污染的树，因为要重置树节点值，所以要借用辅助函数。为了以后查找二叉树结点，要新建私有成员变量head
存储二叉树根节点。  
FindElements函数开始应该把根节点值置为0，左结点传入2 * node->val + 1,右节点传入2 * node->val + 2。递归中依然如此。  
查找就是普通的二叉树查找。
这里遍历过程中可以把所有结点存到某个set里面，后面直接查找set就行，不过这不符合我程序完整性思想。
```c++
struct TreeNode {
    int val;
  	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

class FindElements {
public:
	void helper(TreeNode* node, int val)
	{
		if (node == NULL)
			return;
		node->val = val;
		helper(node->left, 2 * node->val + 1);
		helper(node->right, 2 * node->val + 2);
	}

    FindElements(TreeNode* root) {
		head = root;
		root->val = 0;
		helper(root->left, 1);
		helper(root->right, 2);
    }
    
	bool helper2(TreeNode* node, int&  target)
	{
		if (node == NULL)
			return false;
		if (node->val == target)
			return true;
		return helper2(node->left, target) || helper2(node->right, target);
	}

    bool find(int target) {
		return helper2(head, target);
    }
private:
	TreeNode* head = NULL;
};
```

# 第三道 1262. Greatest Sum Divisible by Three
## 问题描述
Given an array ``nums`` of integers, we need to find the maximum possible sum of elements of the array such that it is divisible by three.
#### Example 1:
```c++
Input: nums = [3,6,5,1,8]
Output: 18
Explanation: Pick numbers 3, 6, 1 and 8 their sum is 18 (maximum sum divisible by 3).
```
#### Example 2:
```c++
Input: nums = [4]
Output: 0
Explanation: Since 4 is not divisible by 3, do not pick any number.
```
#### Example 3:
```c++
Input: nums = [1,2,3,4,4]
Output: 12
Explanation: Pick numbers 1, 3, 4 and 4 their sum is 12 (maximum sum divisible by 3).
```
#### Constraints:
- ``1 <= nums.length <= 4 * 10^4``
- ``1 <= nums[i] <= 10^4``

## 解法
这道题要求出数组中能被3整除的最大子数组和，这里不是限定连续子数组，所有贪心无法解，就只有动态规划了。  
那么首先要解决动态数组如何定义的问题，咋一看好像是定义为dp[i]表示到第i(或者i+1)个位置能被3整除的最大子数组和，但这样定义没有保留除3余1和2的子数组和，后面能被
3整除的和可以由前面除3余1或2加某些数得到。所以dp[i]应该表示除3余i的子数组最大和，(i=0,1,2)。这里建立一维就行了，因为状态方程会不停更新覆盖。  
第二点要解决的就是状态转移方程。对nums中的每个数num除3的余数为num%3。它可以和别的数组成除3余0，1，2的子数组和，假设mod=num%3，那么除3余0的子数组和就是
dp[(3+0-mod)%3] + num,除3余1的子数组和就是dp[(3+1-mod)%3] + num,除3余2的子数组和就是dp[(3+2-mod)%3] + num。注意这里加3的作用是为了防止例如0-mod小于0，反正后面还会余3，就算大于等于3也没影响。然后根据dp大小取大的。  
这里还有初始情况考虑，if (a || mod == 0) dp[0] = max(dp[0], a + num)。因为刚开书动态数组没有加过数为0，就根据mod来更新相关位置。  
最终返回dp[0]。
```c++
int maxSumDivThree(vector<int>& nums)
{
	vector<int> dp(3);
	for (int& num : nums)
	{
		int mod = num % 3;
		int a = dp[(3 + 0 - mod) % 3];
		int b = dp[(3 + 1 - mod) % 3];
		int c = dp[(3 + 2 - mod) % 3];
		if (a || mod == 0)
			dp[0] = max(dp[0], a + num);
		if (b || mod == 1)
			dp[1] = max(dp[1], b + num);
		if (c || mod == 2)
			dp[2] = max(dp[2], c + num);
	}
	return dp[0];
}
```
timecomplexity: O(n)  
space complexity: O(1)  
runtime: 85.71%  
memory usage: 100.00%  

# 第四道 1262. Greatest Sum Divisible by Three
## 问题描述
Storekeeper is a game, in which the player pushes boxes around in a warehouse, trying to get them to target locations.

The game is represented by a grid of size n*m, where each element is a wall, floor or a box.
Your task is move the box 'B' to the target position 'T' under the following rules:
- Player is represented by character 'S' and can move up, down, left, right in the grid if its a floor (empy cell).
- Floor is represented by character '.' that means free cell to walk.
- Wall is represented by character '#' that means obstacle  (impossible to walk there). 
- There is only one box 'B' and one target cell 'T' in the grid.
- The box can be moved to an adjacent free cell by standing next to the box and then moving in the direction of the box. This is a **push**.
- The player cannot walk through the box.

#### Example 1:
![example1](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-11-17/1262-example1.PNG)
```c++
Input: grid = [["#","#","#","#","#","#"],
               ["#","T","#","#","#","#"],
               ["#",".",".","B",".","#"],
               ["#",".","#","#",".","#"],
               ["#",".",".",".","S","#"],
               ["#","#","#","#","#","#"]]
Output: 3
Explanation: We return only the number of times the box is pushed.
```
#### Example 2:
```c++
Input: grid = [["#","#","#","#","#","#"],
               ["#","T","#","#","#","#"],
               ["#",".",".","B",".","#"],
               ["#","#","#","#",".","#"],
               ["#",".",".",".","S","#"],
               ["#","#","#","#","#","#"]]
Output: -1
```
#### Example 3:
```c++
Input: grid = [["#","#","#","#","#","#"],
               ["#","T",".",".","#","#"],
               ["#",".","#","B",".","#"],
               ["#",".",".",".",".","#"],
               ["#",".",".",".","S","#"],
               ["#","#","#","#","#","#"]]
Output: 5
Explanation:  push the box down, left, left, up and up.
```
#### Example 4:
```c++
Input: grid = [["#","#","#","#","#","#","#"],
               ["#","S","#",".","B","T","#"],
               ["#","#","#","#","#","#","#"]]
Output: -1
```
#### Constraints:
- ``1 <= grid.length <= 20``
- ``1 <= grid[i].length <= 20``
- ``grid ``contains only characters ``'.', '#',  'S' , 'T',`` or ``'B'``.
- There is only one character ``'S'``, ``'B'`` and ``'T'`` in the grid.

## 解法
这题属于广度优先搜索的问题。这里用优先队列存储所有可能的状态，把推的次数少的放顶部，**注意这里比较函数是greater，优先队列中greater表示小的在顶部**。用一个map存储所有走过的位置。   
首先要用s，b，t获取人开始的位置，箱子的位置，目标位置的横纵坐标。然后把{s[0], s[1], b[0], b[1], 0}推入队列，第4位的0表示现在推了0次。用一个偏移数组ma[{s[0], s[1], b[0], b[1]}]=1,表示初始位置已经走过了。  
dir_offset={0, -1, 0, 1, 0}存储横纵坐标偏移量，向左移时取x += dir_offset[0], y += dir_offset[1];向上移时取x += dir_offset[1], y += dir_offset[2]...能看出从0到3依次表示向左，上，右，下移动偏移，相当巧妙。  
然后取出队列中移动次数最少的状态，分别向左，上，右，下移动一个位置，遇到'#'表示是墙，continue。如果向某方向移动一下到了箱子的位置，就可以把状态更新。
```c++
int minPushBox(vector<vector<char>>& grid)
{
	vector<int> dir_offset = {0, -1, 0, 1, 0};
	auto cmp = [](const vector<int>& a, const vector<int>& b) {return a[4] > b[4];};
	priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> q(cmp);
	map<vector<int>, int> ma;
	vector<int> s, b, t;
	int m = grid.size(), n = grid[0].size();
	for (int i = 0; i < m; ++i)
	{
		for (int j = 0; j < n; ++j)
		{
			if (grid[i][j] == 'S'){
				s.push_back(i);
				s.push_back(j);
			}
			if (grid[i][j] == 'B'){
				b.push_back(i);
				b.push_back(j);
			}
			if (grid[i][j] == 'T'){
				t.push_back(i);
				t.push_back(j);
			}
		}
	}
	s.push_back(b[0]);
	s.push_back(b[1]);
	s.push_back(0);
	q.push(s);
	ma[{s[0], s[1], b[0], b[1]}] = 1;
	while (!q.empty()){
		vector<int> cur = q.top();
		q.pop();
		if (cur[2] == t[0] && cur[3] == t[1])
			return cur[4];
		for (int i = 0; i < 4; ++i)
		{
			int x = dir_offset[i] + cur[0];
			int y = dir_offset[i + 1] + cur[1];
			if (x < 0 || x == m || y < 0 || y == n)
				continue;
			if (grid[x][y] == '#')
				continue;
			if (x == cur[2] && y == cur[3]){
				int xx = dir_offset[i] + x;
				int yy = dir_offset[i + 1] + y;
				if (xx < 0 || xx == m || yy < 0 || yy == n)
					continue;
				if (grid[xx][yy] == '#')
					continue;
				if (!ma.count({x, y, xx, yy})){
					ma[{x, y, xx, yy}] = 1;
					q.push({x, y, xx, yy, cur[4] + 1});
				}
			}
			else if (!ma.count({x, y, cur[2], cur[3]})){
				ma[{x, y, cur[2], cur[3]}] = 1;
				q.push({x, y, cur[2], cur[3], cur[4]});
			}
		}
	}
	return -1;
}
```
runtime: 25.00%  
memory usage: 100.00%  


# Link
[1260. Shift 2D Grid](https://leetcode.com/contest/weekly-contest-163/problems/shift-2d-grid/)  
[1261. Find Elements in a Contaminated Binary Tree](https://leetcode.com/contest/weekly-contest-163/problems/find-elements-in-a-contaminated-binary-tree/)  
[1262. Greatest Sum Divisible by Three](https://leetcode.com/contest/weekly-contest-163/problems/greatest-sum-divisible-by-three/)  
[1263. Minimum Moves to Move a Box to Their Target Location](https://leetcode.com/contest/weekly-contest-163/problems/minimum-moves-to-move-a-box-to-their-target-location/)  
[推箱子题解](https://leetcode-cn.com/problems/minimum-moves-to-move-a-box-to-their-target-location/solution/c-you-xian-dui-lie-by-war-war-toe/)  

