---
title: "LeetCode 双指针"
date: 2019-07-31T19:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode DFS](https://leetcode-cn.com/tag/two-pointers/)

## 题目

#### [26 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

用两个指针 len 和 i 分别表示没有重复的项的下标与遍历数组的下标，将没有重复的项拷贝到 nums[len] 下然后 ++len 即可。

```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int count = 0, len = 1, n = nums.size();
        if (n == 0)
            return 0;
        for (int i = 1; i < n; ++i) {
            if (nums[i] == nums[i - 1])
                continue;
            nums[len] = nums[i];
            ++len;
        }
        return len;
    }
};
```

#### [80 删除排序数组中的重复项 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array-ii/)

用两个指针 len 和 i 分别表示没有最多重复 2 次的项的下标与遍历数组的下标，将重复数小于等于 1 的项拷贝到 nums[len] 下然后 ++len 即可。

```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int count = 0, len = 1, n = nums.size();
        if (n == 0)
            return 0;
        for (int i = 1; i < nums.size(); ++i) {
            if (nums[i] == nums[i - 1]) {
                if (count > 0)
                    continue;
                ++count;
            }
            else
                count = 0;
            nums[len] = nums[i];
            ++len;
        }
        return len;
    }
};
```

#### [922 按奇偶排序数组 II](https://leetcode-cn.com/problems/sort-array-by-parity-ii/)

用两个下标 i 和 j 分别表示偶数位和奇数位的下标，如果偶数位下标对应的数不是偶数那么将其与奇数位下标对应的数不是奇数的数进行交换。

```c++
class Solution {
public:
    vector<int> sortArrayByParityII(vector<int>& A) {
        for (int i = 0, j = 1; i < A.size(); i += 2)
            if (A[i] % 2 != 0) {
                while (j < A.size() && A[j] % 2 != 0)
                    j += 2;
                swap(A[i], A[j]);
            }
        return A;
    }
};
```

#### [11 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

用两个指针分别表示数组的头和尾，每次将高度较低的元素的下标往中间移动，同时更新结果。

```c++
class Solution {
public:
    int maxArea(vector<int>& h) {
        int i = 0, j = h.size() - 1, res = 0;
        while (i < j) {
            res = max(res, min(h[i], h[j]) * (j - i));
            if (h[i] < h[j])
                ++i;
            else
                --j;
        }
        return res;
    }
};
```

#### [287 寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number/submissions/)

将出现的数字的绝对值 - 1 作为下标，把对应位置的数字乘以 -1 进行标记，因为只有一个数字重复了，所以如果在标记时如果发现对应位置的数字已经是负数则说明出现过相同的下标，返回该数字即可。

```c++
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        for (int i = 0; i < nums.size(); ++i) {
            int index = abs(nums[i]) - 1;
            if (nums[index] < 0)
                return abs(nums[i]);
            nums[index] *= -1;
        }
        return 0;
    }
};
```

#### [75 颜色分类](https://leetcode-cn.com/problems/sort-colors/)

类似于只有两个数的数组排序，只需要用两个变量 idx_0 = 0, idx_2 = n - 1 分别表示两端的下标，将 0 和 2 分别替换到数组的两端，将 1 留在中间即可。

```c++
class Solution {
public:
    void sortColors(vector<int> &nums) {
        int n = nums.size(), idx_0 = 0, idx_2 = n - 1;
        for (int i = 0; i <= idx_2; ++i) {
            if (nums[i] == 2) {
                swap(nums[i], nums[idx_2]);
                --idx_2, --i;
            } else if (nums[i] == 0) {
                swap(nums[i], nums[idx_0]);
                ++idx_0;
            }
        }
    }
};
```

#### [15 三数之和](https://leetcode-cn.com/problems/3sum/)

首先明确两数之和的做法：排序后用两个指针分别从头和尾往中间遍历，根据大小关系移动指针。三数之和无非就是先固定一个数，使得另外两个数之和等于这个数的负数，因此仍然要先对数组进行排序，为了固定一个数需要用一个 for 循环遍历数组，对于其后的所有元素用两数之和的方法进行求和。为了防止出现重复需要在计算两数之和后不断地移动指针直到当前元素与其前/后一个元素不相同。时间复杂度是 O(n ^ 2)，空间复杂度是 O(1)。

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int> &nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        int n = nums.size(), i = 0;
        while (i < n) {
            int j = i + 1, k = n - 1, target = -nums[i];
            while (j < k) {
                if (nums[j] + nums[k] == target) {
                    res.push_back({nums[i], nums[j], nums[k]});
                    do
                        ++j;
                    while (j < k && nums[j] == nums[j - 1]);
                    do
                        --k;
                    while (j < k && nums[k] == nums[k + 1]);
                } else if (nums[j] + nums[k] < target)
                    ++j;
                else
                    --k;
            }
            do
                ++i;
            while (i < n && nums[i] == nums[i - 1]);
        }
        return res;
    }
};
```

#### [424 替换后的最长重复字符](https://leetcode-cn.com/problems/longest-repeating-character-replacement/)

对于一个子串，我们只需要知道这个子串中出现次数最多的字符的出现次数，就可以根据 j - i + 1 - max_count <= k 知道这个子串是否能被替换为重复子串，因此用滑动窗口的方法固定一个子串，如果这个子串满足条件，那么我们将滑动窗口的右端 j 继续往后移动，否则需要将左端往后移动直到这个子串满足条件，j - i + 1 就是可能的最长重复子串的长度。时间复杂度是 O(n)，空间复杂度是 O(1)。

```c++
class Solution {
public:
    int characterReplacement(string s, int k) {
        int i = 0, j = 0, res = 0, n = s.size(), max_count = 0;
        vector<int> count(26, 0);
        while (j < n) {
            ++count[s[j] - 'A'];
            max_count = max(max_count, count[s[j] - 'A']);
            while (j - i + 1 - max_count > k) {
                --count[s[i] - 'A'];
                ++i;
                for (auto &c:count)
                    max_count = max(max_count, c);
            }
            res = max(res, j - i + 1);
            ++j;
        }
        return res;
    }
};
```

#### [1004 最大连续 1 的个数 III](https://leetcode-cn.com/problems/max-consecutive-ones-iii/)

用左右两个指针保证滑动窗口中有小于等于 K 个 0，如果当前位是 1 那么右边的指针继续向后移动，如果当前位是 0 并且已经有 K 个 0，那么左边的指针往右移动直到出现 0，跳过这一位 0，将右边指针的 0 视作 1，更新结果。

```c++
class Solution {
public:
    int longestOnes(vector<int> &A, int K) {
        int i = 0, res = 0;
        for (int j = 0; j < A.size(); ++j) {
            if (A[j] == 0) {
                if (K > 0)
                    --K;
                else {
                    while (A[i] == 1)
                        ++i;
                    ++i;
                }
            }
            res = max(res, j - i + 1);
        }
        return res;
    }
};
```

#### [42 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/submissions/)

可以先将每个位置左边和右边最高的柱子高度都保存下来，再计算两者中较低的减去当前位置的柱子数量得到当前位置能够接住的雨水数量。

```c++
class Solution {
public:
    int trap(vector<int> &height) {
        int n = height.size(), res = 0;
        vector<int> left(n, 0), right(n, 0);
        for (int i = 1; i < n; ++i)
            left[i] = max(left[i - 1], height[i - 1]);
        for (int i = n - 2; i >= 0; --i)
            right[i] = max(right[i + 1], height[i + 1]);
        for (int i = 0; i < n; ++i)
            res += max(0, min(left[i], right[i]) - height[i]);
        return res;
    }
};
```

也可以用两个变量 l_max 和 r_max 分别记录左边和右边到目前为止最高的柱子高度，每次检查较低的一边，能够接住的雨水数量等于 min(l_max, r_max) 减去当前的柱子高度，同时更新柱子的最高高度。

```c++
class Solution {
public:
    int trap(vector<int> &height) {
        int n = height.size(), res = 0, l_max = 0, r_max = 0, i = 0, j = n - 1;
        while (i <= j) {
            if (l_max <= r_max) {
                res += max(0, min(l_max, r_max) - height[i]);
                l_max = max(l_max, height[i]);
                ++i;
            } else {
                res += max(0, min(l_max, r_max) - height[j]);
                r_max = max(r_max, height[j]);
                --j;
            }
        }
        return res;
    }
};
```

#### [632 最小区间](https://leetcode-cn.com/problems/smallest-range/)

比较容易想到的方法是从每个数组的第一个元素开始遍历，使用一个数组 idx 存储每一个数组当前遍历到的元素的下标，每次取这些元素中的最大最小值进行更新，这样做时间复杂度是 O(m * n)，其中 m 是数组的个数，n 是所有元素的个数，但是这样做会 TLE。相较于每次都遍历一遍整个二维数组，我们可以用一个小根堆把所有当前遍历到的元素中的最小值连带其数组下标及其下标保存下来，这样就能每次以 O(1) 的时间复杂度取到所有数组中当前元素的最小值，再用一个变量 max_val 存储所有数组中当前元素的最大值，每次从小根堆 pop 出堆顶元素后，先更新 res 结果数组，然后用这个元素对应下标的后一个下标的值更新 max_val，直到堆顶元素已经是数组的最后一个元素。时间复杂度是 O(m * logn)。

```c++
class Solution {
    struct element {
        int val;
        int vec_idx;
        int idx;

        element(int val, int vec_idx, int idx) : val(val), vec_idx(vec_idx), idx(idx) {}
    };

    struct Compare {
        bool operator()(const element &e1, const element &e2) {
            return e1.val > e2.val;
        }
    };

public:
    vector<int> smallestRange(vector<vector<int>> &nums) {
        int n = nums.size(), max_val = INT_MIN;
        vector<int> res(2, 0);
        res[1] = INT_MAX;
        priority_queue<element, vector<element>, Compare> heap;
        for (int i = 0; i < n; ++i) {
            heap.push(element(nums[i][0], i, 0));
            max_val = max(max_val, nums[i][0]);
        }
        while (true) {
            element e = heap.top();
            heap.pop();
            if (res[1] - res[0] > max_val - e.val)
                res[0] = e.val, res[1] = max_val;
            if (e.idx == nums[e.vec_idx].size() - 1)
                break;
            ++e.idx;
            e.val = nums[e.vec_idx][e.idx];
            heap.push(e);
            max_val = max(max_val, e.val);
        }
        return res;
    }
};
```

#### [76 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

先从左往右找到一个符合条件的字符串，然后用滑动窗口的做法每次在左边去掉一个字符，往右边找到一个未被使用过的对应的字符，如果长度小于之前得到的字符串则更新结果。

```c++
class Solution {
    struct Element {
        int pos;
        char c;

        Element(int pos, char c) : pos(pos), c(c) {}
    };

public:
    string minWindow(string s, string t) {
        string res;
        unordered_map<char, int> count;
        vector<Element> ele;
        unordered_set<int> used;
        for (auto c:t)
            ++count[c];
        for (int i = 0; i < s.size(); ++i)
            if (count.find(s[i]) != count.end())
                ele.push_back(Element(i, s[i]));
        int i = 0, j = 0, n = ele.size(), pos = 0, l = n, start = 0, end = 0;
        while (j < n && !count.empty()) {
            if (count.find(ele[j].c) != count.end()) {
                --count[ele[j].c];
                used.insert(j);
                end = max(end,j);
                if (count[ele[j].c] == 0)
                    count.erase(ele[j].c);
            }
            ++j;
        }
        if (!count.empty())
            return res;
        res = s.substr(ele[i].pos, ele[j - 1].pos - ele[i].pos + 1);
        while (start < n) {
            char target = ele[start].c;
            used.erase(start);
            ++start;
            int k = start;
            while (k < n) {
                if (ele[k].c == target && used.find(k) == used.end()) {
                    used.insert(k);
                    end = min(n - 1, max(end, k));
                    if (res.size() > ele[end].pos - ele[start].pos + 1)
                        res = s.substr(ele[start].pos, ele[end].pos - ele[start].pos + 1);
                    break;
                }
                ++k;
            }
            if (k == n)
                break;
        }
        return res;
    }
};
```

#### [992 K 个不同整数的子数组](https://leetcode-cn.com/problems/subarrays-with-k-different-integers/)

用两个指针 left 和 right 保证滑动窗口内子数组中不同的整数有 K 个，当最左边数的计数大于 1 时代表由 [left, right] 组成的数组和 [left + 1, right] 组成的数组都是符合题意的含有 K 个不同整数的子数组，并且如果 [left + 1, right + 1] 也是符合题意的数组的话那么 [left, right + 1] 也是符合题意的数组，因此 ++acc 并 ++left，当哈希表的 size 等于 K 时将现在 acc 加到结果上去即可。时间复杂度是 O(n)，空间复杂度是 O(n)。

```c++
class Solution {
public:
    int subarraysWithKDistinct(vector<int> &A, int K) {
        unordered_map<int, int> count;
        int acc = 1, res = 0, left = 0;
        for (int right = 0; right < A.size(); ++right) {
            ++count[A[right]];
            while (count.size() > K) {
                --count[A[left]];
                if (count[A[left]] == 0)
                    count.erase(A[left]);
                ++left;
                acc = 1;
            }
            while (count[A[left]] > 1) {
                --count[A[left]];
                ++left;
                ++acc;
            }
            if (count.size() == K)
                res += acc;
        }
        return res;
    }
};
```

#### [239 滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)

用一个类似单调栈的双端队列存储滑动窗口中的元素，当需要 push_back 进来的数大于其前面的数时，不断的将小于它的数 pop_back，这样一来双端队列的 front 位置一定是当前滑动窗口里最大的数，当滑动窗口移动时最左边的数如果等于双端队列中 front 位置的数时则 pop_front，这样一来 front 位置的数仍然是当前滑动窗口里最大的数。这样做时间复杂度是 O(n)，空间复杂度是 O(n)。

```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int> &nums, int k) {
        int n = nums.size();
        deque<int> d;
        vector<int> res;
        if (n == 0)
            return res;
        for (int i = 0; i < k; ++i) {
            while (!d.empty() && d.back() < nums[i])
                d.pop_back();
            d.push_back(nums[i]);
        }
        res.push_back(d.front());
        for (int i = k; i < n; ++i) {
            if (!d.empty() && nums[i - k] == d.front())
                d.pop_front();
            while (!d.empty() && d.back() < nums[i])
                d.pop_back();
            d.push_back(nums[i]);
            res.push_back(d.front());
        }
        return res;
    }
};
```
