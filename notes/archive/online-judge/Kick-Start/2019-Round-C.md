---
title: "Kick Start 2019 Round C"
date: 2019-05-26T23:29:45+10:00
draft: false
categories: ["Kick Start"]
markup: mmark
---

# [Kick Start 2019 Round C](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050ff2)

## [Wiggle Walk (6pts, 12pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050ff2/0000000000150aac)

在一个R * C的矩阵里面移动，遇到已经走过的格子直接跳过。数据保证移动时不会超出给定的矩阵。

### Solution: Simulation

用一个visited数组记录已经走过的格子，遇到走过的格子则直接跳过往后遍历。讲道理这个方法时间复杂度是过不了Hidden Test Set的，但是我也不知道为什么就过了。

- 时间复杂度：O(n^2)
- 空间复杂度：O(n^2)

```C++
// C++
#include <iostream>
#include <cmath>
#include <cstdio>
#include <math.h>
#include <limits>
#include <algorithm>
#include <vector>
#include <stack>
#include <queue>
#include <string>
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

using namespace std;

vector<vector<bool>> visited(50001, vector<bool>(50001, false));

void Forward(char &p, int &r, int &c) {
    if (p == 'E') {
        while (visited[r][c + 1])
            ++c;
        visited[r][++c] = true;
    } else if (p == 'W') {
        while (visited[r][c - 1])
            --c;
        visited[r][--c] = true;
    } else if (p == 'N') {
        while (visited[r - 1][c])
            --r;
        visited[--r][c] = true;
    } else if (p == 'S') {
        while (visited[r + 1][c])
            ++r;
        visited[++r][c] = true;
    }
}

void solve(const int &t) {
    int n, R, C, r, c;
    string str;
    scanf("%d %d %d %d %d", &n, &R, &C, &r, &c);
    cin >> str;
    visited = vector<vector<bool>>(50001, vector<bool>(50001, false));
    visited[r][c] = true;
    for (int i = 0; i < n; ++i)
        Forward(str[i], r, c);
    printf("Case #%d: %d %d\n", t, r, c);
}

int main() {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
        solve(t);
    return 0;
}
```

## [Circuit Board (14pts, 20pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050ff2/0000000000150aae)

在矩阵里找到每一行最大值与最小值不超过K的最大子矩阵。

### Solution: Dynamic Programming

定义一个矩阵len[r][c]来保存格子r, c在r行上符合条件的最长数组。因为题目要求是需要保证每一行的最大值与最小值不超过K，行与行之间是没有关系的，所以对于每一个格子，遍历该列上的所有格子，找到这些格子能够到达的最远的位置，取最小值，用当前的高度乘最远位置就能得到答案。

- 时间复杂度：O(RRC)
- 空间复杂度：O(RC)

```C++
// C++
#include <iostream>
#include <cmath>
#include <cstdio>
#include <math.h>
#include <limits>
#include <algorithm>
#include <vector>
#include <stack>
#include <queue>
#include <string>
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

using namespace std;

void solve(const int &t) {
    int R, C, K;
    scanf("%d %d %d", &R, &C, &K);
    vector<vector<int>> len(301, vector<int>(301, 1)), matrix(301, vector<int>(301));
    for (int r = 0; r < R; ++r)
        for (int c = 0; c < C; ++c)
            scanf("%d", &matrix[r][c]);
    for (int r = 0; r < R; ++r) {
        for (int c = 0; c < C; ++c) {
            int i = c - 1, current_max = matrix[r][c], current_min = matrix[r][c];
            while (i >= 0) {
                current_max = max(current_max, matrix[r][i]);
                current_min = min(current_min, matrix[r][i]);
                if (current_max - current_min > K)
                    break;
                --i;
            }
            len[r][c] = c - i;
        }
    }
    int res = 1;
    for (int r = 0; r < R; ++r) {
        for (int c = 0; c < C; ++c) {
            int min_c = len[r][c];
            for (int line = r; line >= 0; --line) {
                min_c = min(min_c, len[line][c]);
                res = max(res, min_c * (r - line + 1));
            }
        }
    }
    printf("Case #%d: %d\n", t, res);
}

int main() {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
        solve(t);
    return 0;
}
```

## [Catch Some (18pts, 30pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050ff2/0000000000150a0d)

// TODO