---
title: "Kick Start 2019 Round B"
date: 2019-04-21T14:41:22+10:00
draft: false
categories: ["Kick Start"]
markup: mmark
---

# [Kick Start 2019 Round B](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050eda)

## [Building Palindromes (5pts, 12pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050eda/0000000000119866)

判断给定区间内的子字符串是否是回文串。

### Solution: Prefix Sum

判断字符串是否是回文串只需要判断字符串里个数为奇数的字符的数量是否小于等于1，但如果每次都遍历一遍给定的区间肯定会超时，所以我们需要对给定的原始字符串进行预处理，计算出每一个位置的前缀和（从下标为0到下标为i - 1的位置的字符的总数）。这样在查询的时候就只有O(1)的时间复杂度了。

- 时间复杂度：O(N)
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

using namespace std;


int main() {
    int total_test_case_number;
    cin >> total_test_case_number;
    for (int case_number = 1; case_number <= total_test_case_number; ++case_number) {
        int N, Q, m, n, total = 0;
        cin >> N >> Q;
        string words;
        cin >> words;
        vector<vector<int>> odds(N + 1, vector<int>(26));
        for (int i = 1; i <= N; ++i) {
            for (int j = 0; j < 26; ++j)
                odds[i][j] = odds[i - 1][j];
            ++odds[i][words[i - 1] - 'A'];
        }
        for (int q = 0; q < Q; ++q) {
            cin >> m >> n;
            int odd = 0;
            for (int j = 0; j < 26; ++j)
                odd += ((odds[n][j] - odds[m - 1][j]) % 2 != 0);
            total += odd <= 1;
        }
        printf("Case #%d: %d\n", case_number, total);
    }

    return 0;
}
```

## [Energy Stones (17pts, 24pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050eda/00000000001198c3)

在所有物品消耗完之前吃掉，使得能够获得的energy最多，类似于[背包问题](https://zh.wikipedia.org/wiki/%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%98)。

### Solution 1: Dynamic Programming (Visible Test Set)

对于Visible Test Set，吃掉每一件物品所消耗的seconds都是相同的，因此可以简化为一个单纯的背包问题。为了最大化获得的energy（最小化所有物品的总lost），应该先吃掉lost高的物品，所以按照lost排序。

初始状态是dp[index][time] = 0，index = 0, time = 0。状态转移方程是

1. 如果当前物品还有剩余的energy则吃掉当前的物品
```
    if (stones[index].energy > stones[index].lost * time)
        res = max(res, stones[index].energy - stones[index].lost * time + DP(index + 1, time + stones[index].seconds));
```

2. 不吃当前的物品
```
    res = max(res, DP(index + 1, time));
```

- 时间复杂度：O(N * (S * N))
- 空间复杂度：O(N * (S * N))

```C++
//C++
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

struct Stone {
    int seconds;
    int energy;
    int lost;
};
vector<Stone> stones;
vector<vector<int>> dp;
int N;

int DP(int index, int time) {
    if (index >= N)
        return 0;
    int &res = dp[index][time];
    if (res != -1)
        return res;
    if (stones[index].energy > stones[index].lost * time)
        res = max(res, stones[index].energy - stones[index].lost * time + DP(index + 1, time + stones[index].seconds));
    res = max(res, DP(index + 1, time));
    return res;
}

int solve(const int &T) {
    cin >> N;
    stones = vector<Stone>(N);
    for (int i = 0; i < N; ++i)
        cin >> stones[i].seconds >> stones[i].energy >> stones[i].lost;
    dp = vector<vector<int>>(N, vector<int>(stones[0].seconds * N, -1));
    sort(stones.begin(), stones.end(), [](const Stone &s1, const Stone &s2) {
        return s1.lost > s2.lost;
    });
    return DP(0, 0);
}

int main() {
    int T;
    cin >> T;
    for (int t = 1; t <= T; ++t)
        printf("Case #%d: %d\n", t, solve(t));
    return 0;
}
```

### Solution 2: Dynamic Programming (Hidden Test Set)

对于Hidden Test Set，每一件物品消耗的时间并不相同，不能直接使用lost进行排序。对于每两件物品s1和s2，只要满足s1.lost * s2.seconds > s2.lost * s1.seconds就可以保证得到更小的总lost。同时dp数组的大小需要按照每个物品的耗时来计算。

- 时间复杂度：O(N * (S * N))
- 空间复杂度：O(N * (S * N))

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

struct Stone {
    int seconds;
    int energy;
    int lost;
};
vector<Stone> stones;
vector<vector<int>> dp;
int N;

void solve(const int &T) {
    scanf("%d", &N);
    stones = vector<Stone>(N);
    int total_time = 0;
    for (int i = 0; i < N; ++i) {
        scanf("%d %d %d", &stones[i].seconds, &stones[i].energy,&stones[i].lost);
        total_time += stones[i].seconds;
    }
    sort(stones.begin(), stones.end(), [](const Stone &s1, const Stone &s2) {
        return s1.lost * s2.seconds > s2.lost * s1.seconds;
    });
    int res = 0;
    dp = vector<vector<int>>(N + 1, vector<int>(total_time + 1, -1));
    for (int i = 1; i <= N; ++i) {
        dp[i - 1][0] = 0;
        for (int j = 0; j < stones[i - 1].seconds; ++j)
            dp[i][j] = dp[i - 1][j];
        for (int j = stones[i - 1].seconds; j <  dp[i - 1].size(); ++j) {
            dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - stones[i - 1].seconds] + stones[i - 1].energy - (j - stones[i - 1].seconds) * stones[i - 1].lost);
            res = max(res, dp[i][j]);
        }
    }
    printf("Case #%d: %d\n", T, res);
}

int main() {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
        solve(t);
    return 0;
}
```

## [Diverse Subarray (14pts, 28pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050eda/00000000001198c1)

给定最大值S，选取一段连续子数组，使得子数组中相同值的个数不超过S的总数最大化。

### Solution 1: Brute Force (Visible Test Set)

Visible Test Set的数组长度N <= 1000，只需要两层循环做穷举，每次计算符合要求的总数。

- 时间复杂度：O(N^2)
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

using namespace std;


int main() {
    int total_test_case_number;
    cin >> total_test_case_number;
    for (int case_number = 1; case_number <= total_test_case_number; ++case_number) {
        int N, S, res = 0;
        cin >> N >> S;
        vector<int> trinkets(N);
        for (int i = 0; i < N; ++i)
            cin >> trinkets[i];
        for (int j = 0; j < N; ++j) {
            unordered_map<int, int> types;
            for (int i = j; i >= 0; --i) {
                ++types[trinkets[i]];
                int total = 0;
                for (auto t:types) {
                    if (t.second <= S)
                        total += t.second;
                }
                res = max(res, total);
            }
        }

        printf("Case #%d: %d\n", case_number, res);
    }

    return 0;
}
```

### Solution 2: Segment Tree (Hidden Test Set)

// TODO