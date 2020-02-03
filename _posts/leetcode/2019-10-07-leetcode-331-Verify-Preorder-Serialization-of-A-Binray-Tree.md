---
layout: post
title: "Leetcode331-Verify Preorder Serialization of a Binary Tree"
subtitle: 'Leetcode331-stack'
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - stack
---

# 问题描述
One way to serialize a binary tree is to use pre-order traversal. When we encounter a non-null node, we record the node's value. If it is a null node, we record using a sentinel value such as #.  
```c++
     _9_
    /   \
   3     2
  / \   / \
 4   1  #  6
/ \ / \   / \
# # # #   # #  
```
For example, the above binary tree can be serialized to the string "9,3,4,#,#,1,#,#,2,#,6,#,#", where # represents a null node.  
Given a string of comma separated values, verify whether it is a correct preorder traversal serialization of a binary tree. Find an algorithm without reconstructing the tree.
Each comma separated value in the string must be either an integer or a character '#' representing null pointer.
You may assume that the input format is always valid, for example it could never contain two consecutive commas such as "1,,3".
Example 1:
```c++
Input: "9,3,4,#,#,1,#,#,2,#,6,#,#"
Output: true
```
Example 2:
```c++
Input: "1,#"
Output: false
```
Example 2:
```c++
Input: "9,#,#,1"
Output: false
```

# 思路：给定一个字符串，判断是否为二叉树先序遍历序列化字符串。
要点：
1. 先序遍历序列化为满二叉树，叶结点个数比内部结点个数多一个，即"#"比数字多一个；
2. 末尾字符一定为"#"。

# 解法1：
需要把每个字符以逗号','为间隔读入，先解码，利用getline函数。解码完利用cnt记录"#"出现次数，遇"#",cnt加1，否则减1。
如果运行中cnt < 0,说明非法，返回false。运行结束cnt应该等于0且末尾字符为"#"。
```c++
#include <iostream>
#include <vector>
#include <string>
#include <sstream>

using namespace std;

bool isValidSerialization(string preorder) 
{
    istringstream in(preorder);
    int cnt = 0;
    string t = "";
    vector<string> vec;
    while (getline(in, t, ','))
      vec.push_back(t);
    for (int i = 0; i < vec.size() - 1; ++i)
    {
      if (vec[i] == "#"){
        --cnt;
        if (cnt < 0)
          return false;
      }
      else {
        ++cnt;
      }
    }
    return cnt == 0 && vec.back() == "#";
}
```
## 扩展：getline函数两种形式  
在C++中本质上有两种getline函数，一种在头文件<istream>中，是istream类的成员函数。一种在头文件<string>中，是普通函数。
1. 在<istream>中的getline()函数有两种重载形式：
```c++
istream& getline (char* s, streamsize n );
istream& getline (char* s, streamsize n, char delim );
```
作用是： 从istream中读取至多n个字符(包含结束标记符)保存在s对应的数组中。即使还没读够n个字符，
如果遇到delim或 字数达到限制，则读取终止，delim都不会被保存进s对应的数组中。
2. 在<string>中的getline函数有四种重载形式：
```c++
istream& getline (istream&  is, string& str, char delim);
istream& getline (istream&& is, string& str, char delim);
istream& getline (istream&  is, string& str);
istream& getline (istream&& is, string& str);
```
用法和上第一种类似，但是读取的istream是作为参数is传进函数的。读取的字符串保存在string类型的str中。  
is    ：表示一个输入流，例如cin。
str   ：string类型的引用，用来存储输入流中的流信息。
delim ：char类型的变量，所设置的截断字符；在不自定义设置的情况下，遇到’\n’，则终止输入。

# 解法2：
边解码边判断，degree初始化为，表示容忍的多"#"的个数。运行过程中degree为0，表示按道理应该表示正常二叉树的结束。如果后面再次出现非"#"字符，则为false。运行完判断degree是否为0。
```c++
bool isValidSerialization2(string preorder) 
{
	int degree = 1, degree_is_zero = 0;
	istringstream in(preorder);
	string t = "";
	while (getline(in, t, ',')){
		if (t == "#"){
			--degree;
			if (degree == 0)
				degree_is_zero = 1;
		}
		else {
			++degree;
			if (degree_is_zero)
				return false;
		}
	}
	return degree == 0;
}
```

# 解法2：
不解码，某位加','，判断所有字符。初始容忍capacity为1，不是',',continue。否则capacity减1,如果capacity< 0,返回false。
再判断','前是否为"#"，为"#"则继续，不为"#"表示前为数字，capacity加2，其中加1补回前面减小的，多个一个1表示容忍度加1。
```c++
bool isValidSerialization2(string preorder) 
{
	int degree = 1, degree_is_zero = 0;
	istringstream in(preorder);
	string t = "";
	while (getline(in, t, ',')){
		if (t == "#"){
			--degree;
			if (degree == 0)
				degree_is_zero = 1;
		}
		else {
			++degree;
			if (degree_is_zero)
				return false;
		}
	}
	return degree == 0;
}
```