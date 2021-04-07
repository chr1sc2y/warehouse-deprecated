---
title: "LeetCode 排序"
date: 2019-08-17T19:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 排序](https://leetcode-cn.com/tag/sort/)

## 题目

#### [56 合并区间](https://leetcode-cn.com/problems/merge-intervals/)

按照间隔的起始进行排序，判断下一个间隔的起始是否大于前一个间隔的末尾，如果大于的话就把之前的间隔加入结果数组，否则继续扩展当前的间隔。

```c++
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>> &intervals) {
        vector<vector<int>> res;
        int n = intervals.size();
        if (n == 0)
            return res;
        sort(intervals.begin(), intervals.end(), [](vector<int> const &v1, vector<int> const &v2) {
            return v1[0] < v2[0];
        });
        int start = intervals[0][0], end = intervals[0][1];
        for (int i = 1; i < n; ++i) {
            if (intervals[i][0] > end) {
                res.push_back(vector<int>{start, end});
                start = intervals[i][0];
            }
            end = max(end, intervals[i][1]);
        }
        res.push_back(vector<int>{start, end});
        return res;
    }
};
```

#### [179 最大数](https://leetcode-cn.com/problems/largest-number/)

先把数字转换为字符串，然后自定义类似于字典序的排序规则 s1 + s2 > s2 + s1 进行排序。

```c++
class Solution {
public:
    string largestNumber(vector<int> &nums) {
        int i = 0, n = nums.size();
        string res;
        vector<string> strs(n);
        for (i = 0; i < n; ++i)
            strs[i] = to_string(nums[i]);
        sort(strs.begin(), strs.end(), [](const string &s1, const string &s2) {
            return s1 + s2 > s2 + s1;
        });
        for (i = 0; i < n; ++i)
            res += strs[i];
        i = 0;
        while (i < res.size() - 1 && res[i] == '0')
            ++i;
        return res.substr(i);
    }
};
```

#### [324 摆动排序 II](https://leetcode-cn.com/problems/wiggle-sort-ii/)

先对数组进行排序，然后从小到大地将数字间隔着放在新的数组中，这样偶数位上的数字一定比奇数位上的数字大。如果数组长度为偶数，那么初始位置是 n - 2（倒数第二位），否则是 n - 1（最后一位），不从第一位开始是因为有些。这样做时间复杂度是 O(nlogn)，空间复杂度是 O(n)。

```c++
class Solution {
public:
    void wiggleSort(vector<int> &nums) {
        int n = nums.size(), i = n % 2 == 0 ? n - 2 : n - 1;
        vector<int> res(n);
        sort(nums.begin(), nums.end());
        for (int j = 0; j < n; ++j) {
            res[i] = nums[j];
            i -= 2;
            if (i < 0)
                i = n % 2 == 0 ? n - 1 : n - 2;
        }
        nums = res;
    }
};
```

#### [524 通过删除字母匹配到字典里最长单词](https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting/)

用双指针判断字典里的每一个字符串是否满足要求，并比较满足要求的字符串与结果字符串，找到长度最长，字典序最小的返回即可。

```c++
class Solution {
public:
    string findLongestWord(string s, vector<string> &d) {
        string res;
        for (auto &ds:d) {
            int l = 0;
            for (int i = 0; i < s.size(); ++i) {
                if (s[i] == ds[l])
                    ++l;
                if (l == ds.size()) {
                    if (ds.size() > res.size() || (ds.size() == res.size() && ds < res))
                        res = ds;
                    break;
                }
            }
        }
        return res;
    }
};
```

#### [969 煎饼排序](https://leetcode-cn.com/problems/pancake-sorting/)

每次把在 [0, j] 内的最大的数移动到 A[j] 的位置上并 --j，每次需要进行两次翻转，一次是把 [0, j] 内最大的数翻转到 A[0]上，一次是把 A[0] 翻转到 A[j] 上，这样就能保证每次都把最大的数移动到了 [0, j] 的末尾，重复这个过程直到 j 减少为 1，此时只剩 1 个数字，结束循环即可。因为在循环内做了翻转操作，时间复杂度是 O(n^2)，空间复杂度是 O(1)。

```c++
class Solution {
public:
    vector<int> pancakeSort(vector<int> &A) {
        vector<int> res;
        int n = A.size(), j = n;
        while (j > 1) {
            int idx = 0;
            for (int i = 0; i < j; ++i)
                if (A[i] > A[idx])
                    idx = i;
            if (idx != j - 1) {
                if (idx != 0) {
                    reverse(A.begin(), A.begin() + idx + 1);
                    res.push_back(idx + 1);
                }
                reverse(A.begin(), A.begin() + j);
                res.push_back(j);
            }
            --j;
        }
        return res;
    }
};
```

#### [1122 数组的相对排序](https://leetcode-cn.com/problems/relative-sort-array/)

给 arr2 中的数字按照下标赋予权制，按照权制数组进行排序即可。

```c++
class Solution {
public:
    vector<int> relativeSortArray(vector<int>& arr1, vector<int>& arr2) {
        vector<int> auth(1001);
        for (int i = 0; i <= 1000; ++i)
            auth[i] = 1001 + i;
        for (int i = 0; i < arr2.size(); ++i)
            auth[arr2[i]] = i;
        sort(arr1.begin(), arr1.end(), [&](const int &c1, const int &c2) {
            return auth[c1] < auth[c2];
        });
        return arr1;
    }
};
```
