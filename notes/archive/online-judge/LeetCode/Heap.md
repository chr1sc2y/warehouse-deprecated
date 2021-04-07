---
title: "LeetCode 堆"
date: 2019-08-05T19:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 堆](https://leetcode-cn.com/tag/heap/)

## 题目

#### [215 数组中的第 K 个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

最简单的堆的应用。

```c++
class Solution {
public:
    int findKthLargest(vector<int> &nums, int k) {
        priority_queue<int, vector<int>, greater<>> heap;
        for (auto &m:nums) {
            if (heap.size() < k || m > heap.top())
                heap.push(m);
            if (heap.size() > k)
                heap.pop();
        }
        return heap.top();
    }
};
```

#### [347 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/submissions/)

先遍历一次统计出数组中各个元素出现的次数，再用一个大根堆将前 k 个高频元素保存下来，最后再将这些元素依次 pop 出来存入结果数组。时间复杂度是 O(n)，空间复杂度是 O(n)。

```c++
class Solution {
public:
    vector<int> topKFrequent(vector<int> &nums, int k) {
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> freq;
        unordered_map<int, int> count;
        vector<int> res(k, 0);
        for (auto &m:nums)
            ++count[m];
        for (auto &c:count) {
            freq.push(pair<int, int>(c.second, c.first));
            if (freq.size() > k)
                freq.pop();
        }
        for (int i = 0; i < k; ++i) {
            pair<int, int> p = freq.top();
            freq.pop();
            res[i] = p.second;
        }
        return res;
    }
};
```

#### [451 根据字符出现频率排序](https://leetcode-cn.com/problems/sort-characters-by-frequency/)

用一个大根堆保存每个字符出现的频率，然后依次将原字符串覆盖即可。

```c++
class Solution {
    struct Element {
        int asc;
        int num;

        Element() : num(0) {}
    };

    struct Compare {
        bool operator()(Element &e1, Element &e2) {
            return e1.num < e2.num;
        }
    };

public:
    string frequencySort(string s) {
        vector<Element> count(200);
        for (int i = 0; i < 200; ++i)
            count[i].asc = i;
        for (auto c:s)
            ++count[c].num;
        priority_queue<Element, vector<Element>, Compare> heap;
        for (auto &c:count)
            if (c.num != 0)
                heap.push(c);
        int i = 0;
        while (!heap.empty()) {
            auto t = heap.top();
            heap.pop();
            for (int j = 0; j < t.num; ++j)
                s[i++] = t.asc;
        }
        return s;
    }
};
```

#### [692 前 K 个高频单词](https://leetcode-cn.com/problems/top-k-frequent-words/)

用一个 pair<string, int> 或结构体保存每个字符串及其计数的信息，并维护一个小根堆，保证堆里存储前 K 个高频单词，放回堆里的所有元素。

```c++
class Solution {
    struct Comp {
        bool operator()(pair<string, int> const &p1, pair<string, int> const &p2) {
            if (p1.second == p2.second)
                return p1.first < p2.first;
            return p1.second > p2.second;
        }
    };

public:
    vector<string> topKFrequent(vector<string> &words, int k) {
        vector<string> res(k);
        unordered_map<string, int> count;
        priority_queue<pair<string, int>, vector<pair<string, int>>, Comp> heap;
        for (auto &word:words)
            ++count[word];
        for (auto &c:count) {
            heap.push(c);
            if (heap.size() > k)
                heap.pop();
        }
        for (int i = 0; i < k; ++i) {
            res[i] = heap.top().first;
            heap.pop();
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

#### [778 水位上升的泳池中游泳](https://leetcode-cn.com/problems/swim-in-rising-water/)

类似于 AI 中用到的 Greedy Best-First Search 方法。不能用贪心是因为未访问过的路径中可能出现最优解，比如 [[0, 4, 7], [5, 8, 9], [2, 3, 1]] 这个矩阵，如果采用贪心会经过路径 0 -> 4 -> 7 -> 9 -> 1，而实际上最优路径是 0 -> 5 - > 2 -> 3 -> 1，所以需要把之前未经过的节点值保存下来，以便在贪心的过程中遇到当前节点大于之前未经过的节点时重新回到之前的位置继续搜索，保存的方法是用一个小根堆保存所有可能经过的下一步节点，这样就可以每次从堆顶取出下一步的最优值，而搜索的策略则用 DFS，每次尝试四个方向未访问过的节点。时间复杂度是 O(n)。

```c++
class Solution {
    struct Element {
        int val;
        int x, y;

        Element(int val, int x, int y) : val(val), x(x), y(y) {}
    };

    struct Compare {
        bool operator()(Element const &e1, Element const &e2) {
            return e1.val > e2.val;
        }
    };

    int dir[4][2] = {{0, -1}, {-1, 0}, {0, 1}, {1, 0}};
public:
    int swimInWater(vector<vector<int>> &grid) {
        int m = grid.size(), n = m ? grid[0].size() : 0, res = 0;
        if (m == 0)
            return 0;
        priority_queue<Element, vector<Element>, Compare> heap;
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        heap.push(Element(grid[0][0], 0, 0));
        visited[0][0] = true;
        while (!heap.empty()) {
            Element e = heap.top();
            heap.pop();
            res = max(res, e.val);
            if (e.x == m - 1 && e.y == n - 1)
                break;
            for (auto &d:dir) {
                int x = e.x + d[0], y = e.y + d[1];
                if (x >= 0 && x < m && y >= 0 && y < n && !visited[x][y]) {
                    heap.push(Element(grid[x][y], x, y));
                    visited[x][y] = true;
                }
            }
        }
        return res;
    }
};
```

#### [703 数据流中的第 K 大元素](https://leetcode-cn.com/problems/kth-largest-element-in-a-stream/)

维护一个大小为 K 的小根堆，如果新的数比堆顶元素大则将这个数 push 进去并维护堆，当堆中的元素数量超过 K 时 pop 出堆顶元素并维护堆。

```c++
class KthLargest {
    priority_queue<int, vector<int>, greater<>> min_heap;
    int k;
public:
    KthLargest(int k, vector<int> &nums) {
        this->k = k;
        min_heap = priority_queue<int, vector<int>, greater<>>();
        for (auto &m:nums) {
            if (min_heap.size() < k || min_heap.top() < m)
                min_heap.push(m);
            if (min_heap.size() > k)
                min_heap.pop();
        }
    }

    int add(int val) {
        if (min_heap.size() < k || min_heap.top() < val)
            min_heap.push(val);
        if (min_heap.size() > k)
            min_heap.pop();
        return min_heap.top();
    }
};
```

#### [295 数据流的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream/)

最优的做法是用一个大根堆和一个小根堆，前者存数据流的后半部分，前者存数据流的前半部分，这样两个堆的堆顶分别是当前数据流的中间的较大和较小的数，每次去中位数只需要取堆顶元素即可，因此取数的时间复杂度是 O(1)，插入数的时间复杂度是 O(logk)，k 是数据流数据总数的一半。还可以用二分查找加上插入排序的做法，这样做插入的时间复杂度是 O(n)，需要将数据流中大于插入数据的所有数往右移动，查找的时间复杂度是 O(logn)。

```c++
class MedianFinder {
    priority_queue<double> max_heap;
    priority_queue<double, vector<double>, greater<>> min_heap;
public:
    /** initialize your data structure here. */
    MedianFinder() {
        max_heap = priority_queue<double>();
        min_heap = priority_queue<double, vector<double>, greater<>>();
    }

    void addNum(int num) {
        if (max_heap.empty() || max_heap.top() > num)
            max_heap.push(num);
        else
            min_heap.push(num);
        while (max_heap.size() > min_heap.size() + 1) {
            min_heap.push(max_heap.top());
            max_heap.pop();
        }
        while (min_heap.size() > max_heap.size() + 1) {
            max_heap.push(min_heap.top());
            min_heap.pop();
        }
    }

    double findMedian() {
        int ls = max_heap.size(), rs = min_heap.size();
        if ((ls + rs) % 2 == 0)
            return (max_heap.top() + min_heap.top()) * 1.0 / 2;
        return ls > rs ? max_heap.top() : min_heap.top();
    }
};
```