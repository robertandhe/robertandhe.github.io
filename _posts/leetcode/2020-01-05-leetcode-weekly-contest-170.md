---
layout: post
title: "Leetcode Weekly Contest 170"
subtitle: 'Leetcode Weekly Contest 170'
date:       2020-01-05 12:22:00
author: "mudux"
header-style: text
tags:
  - algorithm
  - leetcode
  - string
  - dynamic programming
  - BFS
---

## 1309. Decrypt String from Alphabet to Integer Mapping[easy]
### 问题描述
Given a string ``s`` formed by digits (``'0'`` - ``'9'``) and ``'#'`` . We want to map ``s`` to English lowercase characters as follows:
- Characters (``'a'`` to ``'i'``) are represented by (``'1'`` to ``'9'``) respectively.
- Characters (``'j'`` to ``'z'``) are represented by (``'10#'`` to ``'26#'``) respectively. 

Return the string formed after mapping.
It's guaranteed that a unique mapping will always exist.
#### Example 1:
```c++
Input: s = "10#11#12"
Output: "jkab"
Explanation: "j" -> "10#" , "k" -> "11#" , "a" -> "1" , "b" -> "2".
```
#### Example 2:
```c++
Input: s = "1326#"
Output: "acz"
```
#### Example 3:
```c++
Input: s = "25#"
Output: "y"
```
#### Example 4:
```c++
Input: s = "12345678910#11#12#13#14#15#16#17#18#19#20#21#22#23#24#25#26#"
Output: "abcdefghijklmnopqrstuvwxyz"
```
#### Constraints:
- ``1 <= s.length <= 1000``
- ``s[i]`` only contains digits letters (``'0'``-``'9'``) and ``'#'`` letter.
- ``s`` will be valid string such that mapping is always possible.

### 解法
&emsp;解法挺简单，对每个字符看它后面第二个是否为'#'即可确定它是否和后面相连。
```c++
string freqAlphabets(string s) {
    string res = "";
    int i = 0, n = s.size();
    for (i = 0; i < n - 2; ++i)
    {
        if (s[i + 2] == '#'){
            string t = s.substr(i, 2);
            char c = stoi(t) - 10 + 'j';
            res += c;
            i += 2;
        }
        else {
            res += s[i] - '1' + 'a';
        }
    }
    for (; i < n; ++i)
    {
        res += s[i] - '1' + 'a';
    }
    return res;   
}
```


## 1310. XOR Queries of a Subarray
### 问题描述
Given the array ``arr`` of positive integers and the array ``queries`` where ``queries[i] = [Li, Ri]``, for each query ``i`` compute the XOR of elements from ``Li`` to ``Ri`` (that is, ``arr[Li] xor arr[Li+1] xor ... xor arr[Ri]`` ). Return an array containing the result for the given ``queries``.
#### Example 1:
```c++
Input: arr = [1,3,4,8], queries = [[0,1],[1,2],[0,3],[3,3]]
Output: [2,7,14,8] 
Explanation: 
The binary representation of the elements in the array are:
1 = 0001 
3 = 0011 
4 = 0100 
8 = 1000 
The XOR values for queries are:
[0,1] = 1 xor 3 = 2 
[1,2] = 3 xor 4 = 7 
[0,3] = 1 xor 3 xor 4 xor 8 = 14 
[3,3] = 8
```
#### Example 2:
```c++
Input: arr = [4,8,2,10], queries = [[2,3],[1,3],[0,0],[0,3]]
Output: [8,0,4,4]
```
#### Constraints:
- ``1 <= arr.length <= 3 * 10^4``
- ``1 <= arr[i] <= 10^9``
- ``1 <= queries.length <= 3 * 10^4``
- ``queries[i].length == 2``
- ``0 <= queries[i][0] <= queries[i][1] < arr.length``

### 解法
&emsp;这道题属于动态规划的范围，异或有个性质，任何数与0异或还是本身，相同数异或就是0。所以这里可以利用相当于数组累加和的数组累计异或，从头到尾不停异或，为了处理方便，在开头补个0。
```c++
vector<int> xorQueries(vector<int>& arr, vector<vector<int>>& queries) {
    vector<int> res;
    arr.insert(arr.begin(), 0);
    int n = arr.size();
    vector<int> dp(n, 0);
    int t = 0;
    for (int i = 1; i < n; ++i)
    {
        t ^= arr[i];
        dp[i] = t;
    }
    for (int i = 0; i < queries.size(); ++i)
    {
        res.push_back(dp[queries[i][1] + 1] ^ dp[queries[i][0]]);
    }
    return res;      
}
```

## 5305. Get Watched Videos by Your Friends[medium]
### 问题描述
There are ``n`` people, each person has a unique id between ``0`` and ``n-1``. Given the arrays ``watchedVideos`` and ``friends``, where ``watchedVideos[i]`` and ``friends[i]`` contain the list of watched videos and the list of friends respectively for the person with ``id = i``.

Level **1** of videos are all watched videos by your friends, level **2** of videos are all watched videos by the friends of your friends and so on. In general, the level **k** of videos are all watched videos by people with the shortest path equal to k with you. Given your ``id`` and the ``level`` of videos, return the list of videos ordered by their frequencies (increasing). For videos with the same frequency order them alphabetically from least to greatest. 
#### Example 1:
```c++
Input: watchedVideos = [["A","B"],["C"],["B","C"],["D"]], friends = [[1,2],[0,3],[0,3],[1,2]], id = 0, level = 1
Output: ["B","C"] 
Explanation: 
You have id = 0 (green color in the figure) and your friends are (yellow color in the figure):
Person with id = 1 -> watchedVideos = ["C"] 
Person with id = 2 -> watchedVideos = ["B","C"] 
The frequencies of watchedVideos by your friends are: 
B -> 1 
C -> 2
```
#### Example 2:
```c++
Input: watchedVideos = [["A","B"],["C"],["B","C"],["D"]], friends = [[1,2],[0,3],[0,3],[1,2]], id = 0, level = 1
Output: ["B","C"] 
Explanation: 
You have id = 0 (green color in the figure) and your friends are (yellow color in the figure):
Person with id = 1 -> watchedVideos = ["C"] 
Person with id = 2 -> watchedVideos = ["B","C"] 
The frequencies of watchedVideos by your friends are: 
B -> 1 
C -> 2
```
#### Constraints:
- ``n == watchedVideos.length == friends.length``
- ``2 <= n <= 100``
- ``1 <= watchedVideos[i].length <= 100``
- ``1 <= watchedVideos[i][j].length <= 8``
- ``0 <= friends[i].length < n``
- ``0 <= friends[i][j] < n``
- ``0 <= id < n``
- ``1 <= level < n``
- ``if friends[i] contains j, then friends[j] contains i``

### 解法
&emsp;题目中有句话"by people with the shortest path equal to **k** with you",仅在第k层出现的才计算。这里如果采用dfs可能会提前遇到某些person，导致忽略某些应该考虑的person，所以这里应该是bfs。用visited数组记录已经遇到的person，后面层遇到就可跳过。这里借用queue遍历每一层，对每层中每个人找他的friend，如果没visited过就可计算。
```c++
vector<string> watchedVideosByFriends(vector<vector<string>>& watchedVideos,vector<vector<int>>& friends, int id, int level) 
{
    vector<string> res;
    queue<int> q;
    vector<bool> visited(friends.size(), false);
    unordered_map<string, int> m;
    map<int, vector<string>> mem;
    int size;
    q.push(id);
    visited[id] = true;
    while (!q.empty() && level){
        size = q.size();
        while (size--){
            for (int& i : friends[q.front()])
            {
                if (visited[i])
                    continue;
                visited[i] = true;
                q.push(i);
            }
            q.pop();
        }
        --level;
    }
    while (!q.empty()){
        for (string& s : watchedVideos[q.front()])
            m[s] += 1;
        q.pop();
    }
    for (auto& a : m)
    {
        mem[a.second].push_back(a.first);
    }
    for (auto& a : mem)
    {
        sort(a.second.begin(), a.second.end());
        for (string& s : a.second)
            res.push_back(s);
    }
    return res;
}
```

## 1312. Minimum Insertion Steps to Make a String Palindrome[hard]
### 问题描述
Given a string ``s``. In one step you can insert any character at any index of the string.

Return the *minimum* number of steps to make ``s`` palindrome.

A **Palindrome String** is one that reads the same backward as well as forward.

#### Example 1:
```c++
Input: s = "zzazz"
Output: 0
Explanation: The string "zzazz" is already palindrome we don't need any insertions.
```
#### Example 2:
```c++
Input: s = "mbadm"
Output: 2
Explanation: String can be "mbdadbm" or "mdbabdm".
```
#### Example 3:
```c++
Input: s = "leetcode"
Output: 5
Explanation: Inserting 5 characters the string becomes "leetcodocteel".
```
#### Example 4:
```c++
Input: s = "g"
Output: 0
```
#### Example 5:
```c++
Input: s = "no"
Output: 1
```
#### Contraints:
- ``1 <= s.length <= 500``
- All characters of ``s`` are lower case English letters.

### 解法
```c++
int dp(string& s, int i, int j, vector<vector<int>>& m)
{
    if (i >= j)
        return 0;
    if (m[i][j] != -1)
        return m[i][j];
    return m[i][j] = s[i] == s[j] ? dp(s, i + 1, j - 1, m) : (1 + min(dp(s, i + 1, j, m), dp(s, i, j - 1, m)));
}

int minInsertions(string s)
{
    vector<vector<int>> m;
    m.resize(s.size(), vector<int>(s.size(), -1));
    return dp(s, 0, s.size() - 1, m);
}
```

### reference
[problem 1309](https://leetcode.com/contest/weekly-contest-170/problems/decrypt-string-from-alphabet-to-integer-mapping/)  
[problem 1310](https://leetcode.com/contest/weekly-contest-170/problems/xor-queries-of-a-subarray/)  
[problem 1311](https://leetcode.com/contest/weekly-contest-170/problems/get-watched-videos-by-your-friends/)  
[solution 3](https://leetcode.com/problems/get-watched-videos-by-your-friends/discuss/470695/C%2B%2B-BFS-%2B-Count-Sort)  
[solution 4](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/discuss/470684/C%2B%2B-Simple-DP-(Memoization)-and-Bottom-up-O(n)-Space)