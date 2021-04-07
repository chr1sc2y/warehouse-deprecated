---
title: "LeetCode 数组"
date: 2019-07-04T19:12:25+10:00
draft: true
categories: ["LeetCode"]
---

# [LeetCode 数组](https://leetcode-cn.com/tag/array/)

## 题目

### 1. 常规题

#### [485 最大连续1的个数](https://leetcode-cn.com/problems/max-consecutive-ones/)

依次统计即可。

```c++
class Solution {
public:
    int findMaxConsecutiveOnes(vector<int>& nums) {
        int acc = 0, res = 0;
        for (auto &a:nums) {
            acc += (a == 1 ? 1 : -acc);
            res = max(acc, res);
        }
        return res;
    }
};
```

#### [670 最大交换](https://leetcode-cn.com/problems/maximum-swap/)

给一个非负整数，最多交换一次任意两位，求能得到的最大值。

先把数字转换成字符串方便计算，把每个数字在字符串中最后一次出现的位置记录下来，对每一位找比其大的最大数字，如果这个数字的位置在它后面则交换他们的位置并返回。

```c++
class Solution {
public:
    int maximumSwap(int num) {
        vector<int> pos(10, -1);
        string str = to_string(num);
        int n = str.size();
        for (int i = 0; i < n; ++i)
            pos[str[i] - '0'] = i;
        for (int i = 0; i < n; ++i) {
            for (int j = 9; j > str[i] - '0'; --j) {
                if (pos[j] > i) {
                    swap(str[i], str[pos[j]]);
                    return stoi(str);
                }
            }
        }
        return num;
    }
};
```