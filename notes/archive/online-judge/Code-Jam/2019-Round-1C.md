---
title: "Code Jam 2019 Round 1C"
date: 2019-05-07T22:02:15+10:00
draft: false
categories: ["Code Jam"]
---

# [Code Jam 2019 Round 1C](https://codingcompetitions.withgoogle.com/codejam/round/00000000000516b9)

## [Robot Programming Strategy (10pts, 18pts)](https://codingcompetitions.withgoogle.com/codejam/round/00000000000516b9/0000000000134c90)

已知所有人石头剪刀布的出招顺序，每一轮同时和所有人比赛，找到必胜的策略。

### Solution: Eliminiating

每一轮遍历当前轮次所有人的出招，如果同时有三种情况（R, P, S）则没有必胜策略，直接输出IMPOSSIBLE；否则返回胜利或打平的策略。

对于已经打败过的对手没有必要再考虑其之后的出招，所以用一个defeated数组保存已经打败过的对手以便直接跳过。因为当前轮次有可能超过对手的出招顺序长度，所以要用i % size获取对手当前的出招。

- 时间复杂度：O(A ^ 2)
- 空间复杂度：O(A)

```C++
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

char Decide(const char &R, const char &P, const char &S) {
    if (R && P && S)
        return 'X';
    if (R && P)
        return 'P';
    if (R && S)
        return 'R';
    if (P && S)
        return 'S';
    if (R)
        return 'P';
    if (P)
        return 'S';
    return 'R';
}

bool Defeate(const char &current, const char &opponent) {
    return (current == 'R' && opponent == 'S') || (current == 'S' && opponent == 'P') ||
           (current == 'P' && opponent == 'R');
}

void solve(const int &t) {
    int A;
    scanf("%d", &A);
    int i = 0;
    vector<string> opponent(A);
    vector<bool> defeated(A, false);
    bool R, P, S;
    string res;
    for (int a = 0; a < A; ++a)
        cin >> opponent[a];
    while (true) {
        int current_opponent = 0;
        R = false, P = false, S = false;
        for (int a = 0; a < A; ++a) {
            if (!defeated[a]) {
                ++current_opponent;
                if (opponent[a][i % opponent[a].size()] == 'R')
                    R = true;
                else if (opponent[a][i % opponent[a].size()] == 'P')
                    P = true;
                else
                    S = true;
            }
        }
        if (current_opponent == 0)
            break;
        char result = Decide(R, P, S);
        if (result == 'X') {
            res = "IMPOSSIBLE";
            break;
        }
        res += result;
        for (int a = 0; a < A; ++a) {
            if (!defeated[a] && Defeate(result, opponent[a][i % opponent[a].size()]))
                defeated[a] = true;
        }
        ++i;
    }
    printf("Case #%d: %s\n", t, res.c_str());
}

int main() {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
        solve(t);
    return 0;
}
```

## [Power Arrangers (11pts, 21pts)](https://codingcompetitions.withgoogle.com/codejam/round/00000000000516b9/0000000000134e91)

// TODO

## [Bacterial Tactics (15pts, 25pts)](https://codingcompetitions.withgoogle.com/codejam/round/00000000000516b9/0000000000134cdf)

// TODO