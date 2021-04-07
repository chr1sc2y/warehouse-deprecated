---
title: "LeetCode 广度优先搜索"
date: 2019-07-24T19:12:25+10:00
draft: true
categories: ["LeetCode"]
---

# [LeetCode DFS](https://leetcode-cn.com/tag/breadth-first-search/)

广度优先搜索一般用来求图中的最短路径。

## 题目

### 1. BFS

#### 787 K 站中转内最便宜的航班](https://leetcode-cn.com/problems/cheapest-flights-within-k-stops/)

一共走 K 轮，每轮更新能够到达的点的最小价格，更新时用的数据是上一轮的最小价格，所以要用一个临时的数组来存更新后的最小价格。

```c++
class Solution {
public:
    int findCheapestPrice(int n, vector<vector<int>> &flights, int src, int dst, int K) {
        vector<int> prices(n, INT_MAX);
        prices[src] = 0;
        for (int i = 0; i <= K; ++i) {
            vector<int> u(prices);
            for (auto &f:flights)
                if (prices[f[0]] != INT_MAX)
                    u[f[1]] = min(u[f[1]], prices[f[0]] + f[2]);
            prices = u;
        }
        return prices[dst] == INT_MAX ? -1 : prices[dst];
    }
};
```

#### [1129 颜色交替的最短路径](https://leetcode-cn.com/problems/shortest-path-with-alternating-colors/)

因为要交替地经过两种不同的路径，所以用两个数组 edge[n][2] 来存储两种路径的边，在搜索的时候也需要给每个节点带上之前经过的是哪种路径的信息，才能在之后更换成另一种路径。因为图中可能会存在环，需要用一个数组 visited[n][n][2] 来记录哪些边是已经走过的，或是根据之前已经到达过的距离一定小于当前的距离来判断是否需要继续走下去。

```c++
class Solution {
public:
    vector<int> shortestAlternatingPaths(int n, vector<vector<int>> &red_edges, vector<vector<int>> &blue_edges) {
        vector<int> ret(n);
        vector<vector<int>> res(n,vector<int>(2, INT_MAX));
        res[0][0] = res[0][1] = 0;
        vector<vector<vector<int>>> edge(n, vector<vector<int>>(2, vector<int>()));
        for (auto &r:red_edges)
            edge[r[0]][0].push_back(r[1]);
        for (auto &b:blue_edges)
            edge[b[0]][1].push_back(b[1]);
        queue<pair<int, int>> q;
        q.push(pair<int, int>(0, 0));
        q.push(pair<int, int>(0, 1));
        while (!q.empty()) {
            auto m = q.front();
            q.pop();
            int node = m.first, color = m.second;
            for (auto &e:edge[node][color])
                if (res[e][color ^ 1] > res[node][color] + 1) {
                    res[e][color ^ 1] = res[node][color] + 1;
                    q.push(pair<int, int>(e, color ^ 1));
                }
        }
        for (int i = 0; i < n; ++i) {
            ret[i] = min(res[i][0], res[i][1]);
            if (ret[i] == INT_MAX)
                ret[i] = -1;
        }
        return ret;
    }
};
```

### 2. Dijkstra

#### [743 网络延迟时间](https://leetcode-cn.com/problems/network-delay-time/)

最典型的 Dijkstra。

```c++
class Solution {
public:
    int networkDelayTime(vector<vector<int>> &times, int N, int K) {
        vector<vector<pair<int, int>>> graph(N + 1, vector<pair<int, int>>());
        vector<bool> visited(N + 1, false);
        for (auto &t:times)
            graph[t[0]].push_back(pair<int, int>(t[1], t[2]));
        vector<int> dist(N + 1, INT_MAX);
        dist[K] = 0;
        auto cmp = [](const pair<int, int> &p1, const pair<int, int> &p2) { return p1.second > p2.second; };
        priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(cmp)> heap(cmp);
        heap.push(make_pair(K, 0));
        while (!heap.empty()) {
            pair<int, int> node = heap.top();
            int curr = node.first;
            heap.pop();
            if (visited[curr])
                continue;
            visited[curr] = true;
            for (auto &g:graph[node.first]) {
                int tar = g.first, d = g.second;
                if (!visited[tar] && dist[tar] > dist[curr] + d) {
                    dist[tar] = dist[curr] + d;
                    heap.push(pair<int, int>(tar, dist[tar]));
                }
            }
        }
        int res = *max_element(dist.begin() + 1, dist.end());
        return res == INT_MAX ? -1 : res;
    }
};
```