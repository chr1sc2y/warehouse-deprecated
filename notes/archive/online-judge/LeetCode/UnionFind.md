---
title: "LeetCode 并查集"
date: 2019-07-27T12:12:25+10:00
draft: true
categories: ["LeetCode"]
---

# [LeetCode 并查集](https://leetcode-cn.com/tag/union-find/)

## 题目

#### [1135 最低成本联通所有城市](https://leetcode-cn.com/problems/connecting-cities-with-minimum-cost/submissions/)

首先要保证所有城市均联通，即并查集只有一个根，其次为了使得成本最低，只需要优先考虑权制低的联通边，对权制数组进行从小到大的排序即可。

```c++
class Solution {
    vector<int> uni;
public:
    int minimumCost(int N, vector<vector<int>> &conections) {
        int res = 0;
        sort(conections.begin(), conections.end(), [](vector<int> &v1, vector<int> &v2) { return v1[2] < v2[2]; });
        uni = vector<int>(N + 1, 0);
        for (int i = 1; i <= N; ++i)
            uni[i] = i;
        for (auto &c:conections) {
            int a = Find(c[0]), b = Find(c[1]);
            if (a != b) {
                --N;
                uni[a] = b;
                res += c[2];
            }
        }
        return N == 1 ? res : -1;
    }

    int Find(int i) {
        if (uni[i] != i)
            uni[i] = Find(uni[i]);
        return uni[i];
    }
};
```