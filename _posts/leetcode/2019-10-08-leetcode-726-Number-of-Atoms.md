---
layout: post
title: "Leetcode726-Number of Atoms"
subtitle: 'Leetcode726-stack'
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - stack
  - recursive
---


# 问题描述：
Given a chemical formula (given as a string), return the count of each atom.
An atomic element always starts with an uppercase character, then zero or more lowercase letters, representing the name.
1 or more digits representing the count of that element may follow if the count is greater than 1. If the count is 1, no digits will follow. For example, H2O and H2O2 are possible, but H1O2 is impossible.
Two formulas concatenated together produce another formula. For example, H2O2He3Mg4 is also a formula.
A formula placed in parentheses, and a count (optionally added) is also a formula. For example, (H2O2) and (H2O2)3 are formulas.
Given a formula, output the count of all elements as a string in the following form: the first name (in sorted order), followed by its count (if that count is more than 1), followed by the second name (in sorted order), followed by its count (if that count is more than 1), and so on.  
Example 1:
```c++
Input: 
formula = "H2O"
Output: "H2O"
Explanation: 
The count of elements are {'H': 2, 'O': 1}.
```
Example 2:
```c++
Input: 
formula = "Mg(OH)2"
Output: "H2MgO2"
Explanation: 
The count of elements are {'H': 2, 'Mg': 1, 'O': 2}.
```
Example 3:
```c++
Input: 
formula = "K4(ON(SO3)2)2"
Output: "K4N2O14S4"
Explanation: 
The count of elements are {'K': 4, 'N': 2, 'O': 14, 'S': 4}.
```
Note:
- All atom names consist of lowercase letters, except for the first character which is uppercase.
- The length of formula will be in the range [1, 1000].
- ``formula`` will only consist of letters, digits, and round parentheses, and is a valid formula as defined in the problem.

# 思路：
题目给定一个化学分子式，要求统计每个元素的个数，并且按照元素字母次序排列。
要点：
1. 单原子后面不跟1；
2. 原子均以大写字母开头，后面接若干个小写字母；
3. 括号后面一定有数字；
4. 记录元素个数用哈希表，判断是否为小写字母用islower()函数，判断是否为数字用isdigit()函数；
5. 按次序统计个数简单，重点在于处理括号，适合递归求解。

# 解法1：递归
用有序哈希表存储元素个数，键为原子，值为出现个数。遇到左括号'('进入递归，直到遇到右括号')'。括号内每个元素个数应乘以右括号后数字。
对只有一个原子的元素，后面不跟数字，人为补1。
```c++
map<string, int> parse(string& str, int& pos)
{
	map<string, int> res;
	int n = str.size();
	while (pos < n){
		if (str[pos] == '('){
			++pos;
			for (auto a : parse(str, pos))
				res[a.first] += a.second;
		}
		else if (str[pos] == ')'){
			int i = ++pos;
			while (pos < n && isdigit(str[pos]))
				++pos;
			string elem = str.substr(i, pos - i);
			int multiple = stoi(elem);
			for (auto&a : res)
			{
				a.second *= multiple;
			}
			return res;
		}
		else {
			int i = pos++;
			while (pos < n && islower(str[pos]))
				++pos;
			string t = str.substr(i, pos - i);
			i = pos;
			while (pos < n && isdigit(str[pos]))
				++pos;
			string elem = str.substr(i, pos - i);
			res[t] += (elem.empty() ? 1 : stoi(elem));		
		}
	}
	return res;
}

string countOfAtoms(string formula)
{
	map<string, int> m;
	int pos = 0;
	m = parse(formula, pos);
	string res = "";
	for (auto& a : m)
	{
		res += (a.first  + (a.second == 1 ? "" : to_string(a.second)));
	}
	return res;
}
```
# 解法2：迭代
用迭代解法避免递归。遇到左括号时当前哈希表入栈，move清空哈希表，存入栈顶，然后在括号内统计各原子数目，直到右括号，哈希表内各数字乘以括号后倍数。当前哈希表加入栈顶，最后把栈顶哈希表弹出到cur。
```c++
string countOfAtoms(string formula)
{
	string res = "";
	stack<map<string, int>> st;
	map<string, int> cur;
	int pos = 0, n = formula.size();
	while (pos < n){
		if (formula[pos] == '('){
			st.push(move(cur));
			++pos;
		}
		else if (formula[pos] == ')'){
			int i = ++pos;
			while(pos < n && isdigit(formula[pos])) 
				++pos;
			string elem = formula.substr(i, pos - i);
			int multiple = elem.empty() ? 1 : stoi(elem);
			for (auto& a : cur)
				st.top()[a.first] += a.second * multiple;
			cur = move(st.top());
			st.pop();
		}
		else {
			int i = pos++;
			while (pos < n && islower(formula[pos]))
				++pos;
			string elem = formula.substr(i, pos - i);
			i = pos;
			while (pos < n && isdigit(formula[pos]))
				++pos;
			string elem2 = formula.substr(i, pos - i);
			int t = elem2.empty() ? 1 : stoi(elem2);
			cur[elem] += t;
		}
	}
	for (auto& a : cur)
	{
		res += (a.first + (a.second == 1 ? "" : to_string(a.second)));
	}
	return res;
}
```