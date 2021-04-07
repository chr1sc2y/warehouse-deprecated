---
title: "Kick Start 2019 Round A"
date: 2019-03-26T14:25:36+11:00
draft: false
categories: ["Kick Start"]
markup: mmark
---

# [Kick Start 2019 Round A](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01)

## [Training (7pts, 13pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01/00000000000698d6)

一共有N个人，从中选P个人，计算这P个人中 skill rating 的最大值与其他人的 skill rating 的差值之和。

$$ \sum_{i}^{j} max(rating) - rating[i] $$

### Solution: Sort + Prefix Sum

先对数组排序，然后在长度为N的有序数组中遍历长为P的所有连续子数组，计算子数组中的最大值与其他值的差值之和。

$$ \sum_{i}^{j - 1} rating[j] - rating[i] $$

如果直接遍历长为P的子数组会浪费很多时间，可以将上面的公式简化为如下。

$$ \sum_{i}^{j - 1} rating[j] - rating[i] = rating[j] * (j - 1 - i) - \sum_{i}^{j - 1} rating[i] $$

为了避免重复计算 $$ \sum_{i}^{j - 1} rating[i] $$，可以用一个长为N+1的数组将原始数组的前缀和保存下来，这样每次直接计算 prefix[j] - prefix[i] 就能得到 $$ \sum_{i}^{j - 1} rating[i] $$ 了，时间复杂度是O(N)。

- 时间复杂度：O(NlogN)
- 空间复杂度：O(N)

```C++
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    int c = 1;
    int T;
    std::cin >> T;
    for (int t = 0; t < T; ++t) {
        int N, P;
        int min_hour = 1000000000;
        std::cin >> N >> P;
        std::vector<int> rating(N, 0);
        for (int n = 0; n < N; ++n)
            std::cin >> rating[n];
        std::sort(rating.begin(), rating.end());
        
        std::vector<int> prefix(N + 1, 0);
        for (int i = 0; i < N; ++i)
            prefix[i + 1] = prefix[i] + rating[i];

        for (int i = P - 1; i < N; ++i) {
            int sum = rating[i] * (P - 1) - (prefix[i] - prefix[i - P + 1]);
            min_hour = std::max(0, std::min(min_hour, sum));
        }

        std::cout << "Case #" + std::to_string(c) + ": " + std::to_string(min_hour) << std::endl;
        ++c;
    }
    return 0;
}
```

## [Parcels (15pts, 20pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01/000000000006987d)

一道中等难度的BFS，DFS，二分查找的题。需要先计算出图中的曼哈顿距离，再找到一个最优的位置使得其他点离这些点的距离最短。

### Solution #1: Manhattan Distance

曼哈顿距离可以用公式 $$ |r1 - r2| + |c1 - c2| $$ 计算得出，因此可以直接使用两层循环来计算出曼哈顿距离。

第一步对于每个已经存在的 delivery office，计算出其距离所有点的曼哈顿距离，得到此时的最大运输时间 max_time，时间复杂度是O(RC^2)；

第二步依次遍历所有可能的点，计算出在该点添加 delivery office 后的图中的最大运输时间 curr_max_time，与 max_time 比较取最小值，时间复杂度是O(RC^2)。

- 时间复杂度：O(RC^2)
- 空间复杂度：O(RC)

```C++
#include <iostream>
#include <cmath>
#include <queue>
#include <vector>
#include <string>
#include <limits.h>
#include <algorithm>

int main() {
    int num_case = 1;
    int T;
    std::cin >> T;
    for (int t = 0; t < T; ++t) {
        int R, C;
        std::cin >> R >> C;
        std::vector<std::vector<int>> grid(R, std::vector<int>(C, INT_MAX));
        for (int i = 0; i < R; ++i) {
            std::string temp_in;
            std::cin >> temp_in;
            for (int j = 0; j < C; ++j) {
                char &temp = temp_in[j];
                if (temp == '1') {
                    for (int x = 0; x < R; ++x) {
                        for (int y = 0; y < C; ++y) {
                            grid[x][y] = std::min(grid[x][y], abs(i - x) + abs(j - y));
                        }
                    }
                }
            }
        }

        int max_time = 0;
        for (int i = 0; i < R; ++i) {
            for (int j = 0; j < C; ++j) {
                max_time = std::max(max_time, grid[i][j]);
            }
        }

        if (max_time == 0) {
            std::cout << "Case #" + std::to_string(num_case) + ": 0" << std::endl;
            ++num_case;
            continue;
        }

        for (int i = 0; i < R; ++i) {
            for (int j = 0; j < C; ++j) {
                if (grid[i][j] != 0) {
                    int curr_max_time = 0;
                    for (int x = 0; x < R; ++x) {
                        for (int y = 0; y < C; ++y) {
                            curr_max_time = std::max(curr_max_time, std::min(grid[x][y], abs(i - x) + abs(j - y)));
                        }
                    }
                    max_time = std::min(max_time, curr_max_time);
                }
            }
        }

        std::cout << "Case #" + std::to_string(num_case) + ": " + std::to_string(max_time) << std::endl;
        ++num_case;
    }
    return 0;
}
```

### Solution #2: Breadth-First-Search

对于第一步来说，可以将所有已经存在的 delivery office 保存在一个队列中，然后使用BFS直接计算出最短的曼哈顿距离，得到此时的最大运输时间 max_time，时间复杂度是O(RC)；

第二步与第一步类似，也可以通过BFS计算出在所有可能的点增加 delivery office 后的最大运输时间，时间复杂度是O(RC^2)。

- 时间复杂度：O(RC^2)
- 空间复杂度：O(RC)

```C++
#include <iostream>
#include <cmath>
#include <queue>
#include <vector>
#include <string>
#include <limits.h>
#include <algorithm>

int dir_x[4] = {-1, 0, 1, 0};
int dir_y[4] = {0, -1, 0, 1};

int BFS0(const int &R, const int &C, int x, int y,
         std::vector<std::vector<int>> &grid, std::queue<std::pair<int, int>> &que) {
    int max_time = 0;
    int size = que.size(), degree = 1;
    while (!que.empty()) {
        x = que.front().first, y = que.front().second;
        que.pop();
        max_time = std::max(max_time, grid[x][y]);
        for (int i = 0; i < 4; ++i) {
            int new_x = x + dir_x[i], new_y = y + dir_y[i];
            if (new_x >= 0 && new_x < R && new_y >= 0 && new_y < C && grid[new_x][new_y] == -1) {
                que.push(std::pair<int, int>(new_x, new_y));
                grid[new_x][new_y] = degree;
            }
        }
        --size;
        if (size == 0) {
            size = que.size();
            ++degree;
        }
    }
    return max_time;
}

int BFS1(const int &R, const int &C, int x, int y,
         std::vector<std::vector<int>> &grid, std::queue<std::pair<int, int>> &que) {
    std::vector<std::vector<bool>> visited(R, std::vector<bool>(C, false));
    visited[x][y] = true;
    int max_time = INT_MIN;
    int size = 1, degree = 0;
    while (!que.empty()) {
        x = que.front().first, y = que.front().second;
        que.pop();
        max_time = std::max(max_time, std::min(grid[x][y], degree));
        for (int i = 0; i < 4; ++i) {
            int new_x = x + dir_x[i], new_y = y + dir_y[i];
            if (new_x >= 0 && new_x < R && new_y >= 0 && new_y < C && !visited[new_x][new_y]) {
                que.push(std::pair<int, int>(new_x, new_y));
                visited[new_x][new_y] = true;
            }
        }
        --size;
        if (size == 0) {
            size = que.size();
            ++degree;
        }
    }
    return max_time;
}

int main() {
    int num_case = 1;
    int T;
    std::cin >> T;
    for (int t = 0; t < T; ++t) {
        int R, C;
        std::cin >> R >> C;
        std::vector<std::vector<int>> grid(R, std::vector<int>(C, -1));
        std::queue<std::pair<int, int>> que;
        for (int i = 0; i < R; ++i) {
            std::string temp_in;
            std::cin >> temp_in;
            for (int j = 0; j < C; ++j) {
                char &temp = temp_in[j];
                if (temp == '1') {
                    que.push(std::pair<int, int>(i, j));
                    grid[i][j] = 0;
                }
            }
        }
        int size = que.size();
        if (size == R * C) {
            std::cout << "Case #" + std::to_string(num_case) + ": 0" << std::endl;
            ++num_case;
            continue;
        }

        // BFS for existing delivery offices
        int max_time = BFS0(R, C, 0, 0, grid, que);

        // BFS for all possible positions
        for (int i = 0; i < R; ++i) {
            for (int j = 0; j < C; ++j) {
                if (grid[i][j] != 0) {
                    que = std::queue<std::pair<int, int>>();
                    que.push(std::pair<int, int>(i, j));
                    int curr_max_time = BFS1(R, C, i, j, grid, que);
                    max_time = std::min(max_time, curr_max_time);
                }
            }
        }

        std::cout << "Case #" + std::to_string(num_case) + ": " + std::to_string(max_time) << std::endl;
        ++num_case;
    }
    return 0;
}
```

### Solution #3: Binary Search

前面的方法对每个可能的点都进行了搜索，时间复杂度达到了平方级别，因此不能通过 Hidden Test Set。因为题目需要求满足要求的最小值，自然容易想到使用二分法来求下界。

对于一个给定的mid值，如果能够添加新的 delivery office 使得最大运输时间小于等于mid，那么可能有小于 mid 的值 (1 ... k-1) 使得结论成立；如果给定的mid值不能使结论成立，那么大于等于 mid 的所有值 (k ... INT_MAX) 都不能使结论成立。

为了确认图中的点到 delivery office 的最短距离小于 mid，可以将图旋转45度，使用 i + j 和 i - j 分别作为左下和右上两条对角线的值来计算将 mid 作为最大运输时间时能否完成运输。

$$ distance((x1，y1)，(x2，y2))= \max (abs(x1 + y1  - (x2 + y2))，abs(x1  -  y1  - (x2  -  y2))) $$

- 时间复杂度：O(RClog(R+C))
- 空间复杂度：O(RC)

```C++
#include <iostream>
#include <cmath>
#include <queue>
#include <vector>
#include <string>
#include <limits.h>
#include <algorithm>

int dir_x[4] = {-1, 0, 1, 0};
int dir_y[4] = {0, -1, 0, 1};
int R, C;
std::vector<std::vector<int>> grid;

int BFS(int x, int y, std::queue<std::pair<int, int>> &que) {
    int max_time = 0;
    int size = que.size(), time = 1;
    while (!que.empty()) {
        x = que.front().first, y = que.front().second;
        que.pop();
        max_time = std::max(max_time, grid[x][y]);
        for (int i = 0; i < 4; ++i) {
            int new_x = x + dir_x[i], new_y = y + dir_y[i];
            if (new_x >= 0 && new_x < R && new_y >= 0 && new_y < C && grid[new_x][new_y] == -1) {
                que.push(std::pair<int, int>(new_x, new_y));
                grid[new_x][new_y] = time;
            }
        }
        --size;
        if (size == 0) {
            size = que.size();
            ++time;
        }
    }
    return max_time;
}

bool CanDeliver(int &max_time) {
    bool deliver = true;
    int left_low = INT_MAX, left_high = INT_MIN, right_low = INT_MAX, right_high = INT_MIN;
    for (int i = 0; i < R; ++i) {
        for (int j = 0; j < C; ++j) {
            if (grid[i][j] > max_time) {
                deliver = false;
                left_low = std::min(left_low, i + j + max_time);
                left_high = std::max(left_high, i + j - max_time);
                right_low = std::min(right_low, i - j + max_time);
                right_high = std::max(right_high, i - j - max_time);
            }
        }
    }
    if (deliver)
        return true;
    for (int i = 0; i < R; ++i) {
        for (int j = 0; j < C; ++j) {
            int left = i + j, right = i - j;
            if (left_high <= left && left <= left_low && right_high <= right && right <= right_low)
                return true;
        }
    }
    return false;
}

int main() {
    int num_case = 1;
    int T;
    std::cin >> T;
    for (int t = 0; t < T; ++t) {
        std::cin >> R >> C;
        grid = std::vector<std::vector<int>>(R, std::vector<int>(C, -1));
        std::queue<std::pair<int, int>> que;
        for (int i = 0; i < R; ++i) {
            std::string temp_in;
            std::cin >> temp_in;
            for (int j = 0; j < C; ++j) {
                char &temp = temp_in[j];
                if (temp == '1') {
                    que.push(std::pair<int, int>(i, j));
                    grid[i][j] = 0;
                }
            }
        }
        int size = que.size();
        if (size == R * C) {
            std::cout << "Case #" + std::to_string(num_case) + ": 0" << std::endl;
            ++num_case;
            continue;
        }

        // BFS
        int max_time = BFS(0, 0, que);

        // Binary Search
        int lowest_time = 0, highest_time = INT_MAX;
        while (lowest_time < highest_time) {
            int mid_time = lowest_time + ((highest_time - lowest_time) >> 1);
            if (CanDeliver(mid_time))
                highest_time = mid_time;
            else
                lowest_time = mid_time + 1;
        }

        std::cout << "Case #" + std::to_string(num_case) + ": " + std::to_string(highest_time) << std::endl;
        ++num_case;
    }
    return 0;
}
```

## [Contention (18pts, 27pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01/0000000000069881)

// TODO