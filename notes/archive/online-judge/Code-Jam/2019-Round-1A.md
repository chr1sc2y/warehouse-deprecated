---
title: "Code Jam 2019 Round 1A"
date: 2019-04-13T15:28:11+10:00
draft: false
categories: ["Code Jam"]
---

# [Code Jam 2019 Round 1A](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051635)

## [Pylons (8pts, 23pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051635/0000000000104e03)

在m*n的网格里移动，每次移动后的位置不能与之前的位置在同一行/列/对角线上。

### Solution: BackTracking

类似于八皇后问题，不过每次的限制条件只和上一个位置有关，可以用回溯解决。

- 时间复杂度：O(m^2 * n^2)
- 空间复杂度：O(m * n)

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

int m, n;

bool BackTracking(int t, int i, int j, vector<vector<bool>> &visited, vector<vector<int>> &res) {
    visited[i][j] = true;
    res[t] = {i, j};
    if (t + 1 == m * n)
        return true;
    for (int x = 0; x < m; ++x) {
        for (int y = 0; y < n; ++y) {
            int r = (x + i) % m, c = (y + j) % n;
            if (!visited[r][c] && r != i && c != j && r + c != i + j && r - c != i - j &&
                BackTracking(t + 1, r, c, visited, res))
                return true;
        }
    }
    visited[i][j] = false;
    return false;
}

int main() {
    int total_test_case_number;
    cin >> total_test_case_number;
    for (int case_number = 1; case_number <= total_test_case_number; ++case_number) {
        bool rev = false;
        cin >> m >> n;
        if (m > n) {
            rev = true;
            swap(n, m);
        }
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        vector<vector<int>> res(m * n, vector<int>());
        printf("Case #%d: ", case_number);
        if (BackTracking(0, 0, 0, visited, res)) {
            printf("POSSIBLE\n");
            for (int i = 0; i < m * n; ++i)
                if (!rev)
                    printf("%d %d\n", res[i][0] + 1, res[i][1] + 1);
                else
                    printf("%d %d\n", res[i][1] + 1, res[i][0] + 1);
        } else
            printf("IMPOSSIBLE\n");
    }

    return 0;
}
```

## [Alien Rhyme (10pts, 27pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051635/0000000000104e05)

找到后缀相同的一对单词，后缀的长度可以自己定义，其他单词的后缀不能与这一对相同，使得这样的单词对最多。

### Solution: Suffix

先翻转每一个单词（如果不翻转的话取substr的时候就从中间开始取到最后，翻转的话只需要取前面m个字母）。从最长的单词长度依次递减，取每一个单词的后缀，如果两个单词有相同的后缀则把这两个单词去掉，结果+2。因为是从最长的单词长度开始依次取后缀，可以保证后缀相同的单词不被漏掉。

- 时间复杂度：O(N * m)
    - m：最长的单词长度
    - N：单词个数
- 空间复杂度：O(N)

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

#include "Print.h"

using namespace std;


int main() {
    int total_test_case_number;
    cin >> total_test_case_number;
    for (int case_number = 1; case_number <= total_test_case_number; ++case_number) {
        int N, sum = 0, max_len = 0;
        cin >> N;
        vector<string> words(N);
        for (int m = 0; m < N; ++m) {
            cin >> words[m];
            max_len = max(max_len, static_cast<int>(words[m].size()));
            reverse(words[m].begin(), words[m].end());
        }
        unordered_map<string, int> status;
        vector<bool> visited(N, false);
        int total = 0;
        for (int len = max_len; len > 0; --len) {
            for (int m = 0; m < N; ++m) {
                if (visited[m] || words[m].size() < len)
                    continue;
                string str = words[m].substr(0, len);
                if (status.find(str) != status.end()) {
                    if (status[str] != -1) {
                        sum += 2;
                        visited[status[str]] = true, visited[m] = true;
                        status[str] = -1;
                    }
                } else
                    status[str] = m;
            }
        }
        printf("Case #%d: %d\n", case_number, sum);
    }

    return 0;
}
```

## [Golf Gophers (11pts, 21pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051635/0000000000104f1a)

// TODO