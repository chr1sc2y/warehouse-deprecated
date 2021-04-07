---
title: "LeetCode 栈"
date: 2019-08-05T19:12:25+10:00
draft: true
categories: ["LeetCode"]
---

# [LeetCode 栈](https://leetcode-cn.com/tag/stack/)

## 题目

#### [739 每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

反向遍历数组，每次往后找大于当天温度的第一个数，但这样做会浪费时间做重复的遍历；如果某一天的温度 T[j] 大于当天的温度 T[i]，那么在第 j 天之后的温度中比 T[j] 小的都是没有用的，只需要将这些天数删掉即可，保留 T[j] 以及比 T[j] 大的即可，可以用一个单调栈来实现。

```c++
class Solution {
public:
    vector<int> dailyTemperatures(vector<int> &T) {
        int n = T.size();
        stack<int> days;
        vector<int> ans(n);
        for (int i = n - 1; i >= 0; --i) {
            while (!days.empty() && T[i] >= T[days.top()])
                days.pop();
            ans[i] = days.empty() ? 0 : days.top() - i;
            days.push(i);
        }
        return ans;
    }
};
```